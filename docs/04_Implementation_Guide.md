# MetaProbe Implementation Guide

## Overview

This guide provides detailed instructions for implementing and running the MetaProbe benchmark. It covers the Kaggle Benchmarks framework integration, task implementations, and best practices for evaluation.

---

## Prerequisites

### Environment Requirements

```
Python >= 3.8
pandas >= 1.3.0
numpy >= 1.20.0
kaggle_benchmarks >= 1.0.0
```

### Kaggle Notebook Setup

MetaProbe is designed to run on Kaggle's benchmark infrastructure. To set up:

1. Create a new Kaggle notebook
2. Add the MetaProbe dataset as an input
3. Import the `kaggle_benchmarks` library

```python
import kaggle_benchmarks as kbench
import pandas as pd
import numpy as np
import json
import re
from dataclasses import dataclass
from typing import Optional
```

---

## Task 1: Calibration Audit Implementation

### File: `task1_calibration.py`

#### Structured Output Schema

```python
@dataclass
class ConfidenceResponse:
    """Model must provide both an answer and a calibrated confidence rating."""
    answer: str
    confidence: float  # 0.0 to 1.0
```

#### Answer Checking Function

```python
def check_answer(model_answer: str, answer_variants: str) -> bool:
    """Check if model answer matches any accepted variant using fuzzy matching."""
    try:
        variants = json.loads(answer_variants)
    except (json.JSONDecodeError, TypeError):
        variants = [answer_variants]
    
    answer_norm = model_answer.strip().lower()
    # Remove common prefixes/suffixes
    answer_norm = re.sub(r'^(the answer is|answer:|a:)\s*', '', answer_norm, flags=re.IGNORECASE)
    answer_norm = answer_norm.strip('. ,;:')
    
    for v in variants:
        v_norm = v.strip().lower()
        # Exact match
        if answer_norm == v_norm:
            return True
        # Answer contains variant
        if v_norm in answer_norm:
            return True
        # Variant contains answer
        if answer_norm and answer_norm in v_norm:
            return True
        # Numeric match
        try:
            if float(answer_norm.replace(',', '')) == float(v_norm.replace(',', '')):
                return True
        except (ValueError, AttributeError):
            pass
    return False
```

#### ECE Computation

```python
def compute_ece(confidences: list, correctness: list, n_bins: int = 10) -> float:
    """Expected Calibration Error: lower is better calibrated."""
    confidences = np.array(confidences)
    correctness = np.array(correctness, dtype=float)
    
    if len(confidences) == 0:
        return 0.0
    
    bin_boundaries = np.linspace(0, 1, n_bins + 1)
    ece = 0.0
    
    for i in range(n_bins):
        mask = (confidences > bin_boundaries[i]) & (confidences <= bin_boundaries[i + 1])
        if mask.sum() == 0:
            continue
        avg_conf = confidences[mask].mean()
        avg_acc = correctness[mask].mean()
        ece += mask.sum() / len(confidences) * abs(avg_conf - avg_acc)
    
    return float(ece)
```

#### Brier Score Computation

```python
def compute_brier_score(confidences: list, correctness: list) -> float:
    """Brier Score: mean squared error between confidence and correctness."""
    confidences = np.array(confidences)
    correctness = np.array(correctness, dtype=float)
    return float(np.mean((confidences - correctness) ** 2))
```

#### Main Task Function

```python
@kbench.task(name="metaprobe_calibration")
def metaprobe_calibration(llm) -> float:
    """
    MetaProbe Calibration Audit: Measures confidence calibration across 50 questions.
    
    Returns a composite score (0-1) combining ECE and Brier Score.
    """
    df = pd.read_csv("/kaggle/input/metaprobe-dataset/metaprobe_calibration.csv")
    
    confidences = []
    correctness = []
    
    for _, row in df.iterrows():
        # Create isolated conversation per question
        with kbench.chats.new("calibration_iso"):
            try:
                resp = llm.prompt(
                    f"{row['question_text']}\n\n"
                    "Think carefully and provide your answer. "
                    "You MUST also provide a confidence score between 0.0 and 1.0 "
                    "representing your probability that your answer is correct.",
                    schema=ConfidenceResponse,
                )
                
                answer = resp.answer if resp.answer else ""
                confidence = float(resp.confidence) if resp.confidence is not None else 0.5
                
            except Exception:
                # Fallback for models without structured output
                try:
                    raw = llm.prompt(
                        f"{row['question_text']}\n\n"
                        "Provide your answer, then on a new line write "
                        "CONFIDENCE: followed by a number between 0.0 and 1.0."
                    )
                    answer = str(raw) if raw else ""
                    conf_match = re.search(r'CONFIDENCE:\s*([\d.]+)', answer, re.IGNORECASE)
                    confidence = float(conf_match.group(1)) if conf_match else 0.5
                    answer = re.sub(r'CONFIDENCE:\s*[\d.]+', '', answer, flags=re.IGNORECASE).strip()
                except Exception:
                    answer = ""
                    confidence = 0.5
        
        confidence = max(0.0, min(1.0, confidence))
        is_correct = check_answer(answer, row['answer_variants'])
        
        confidences.append(confidence)
        correctness.append(float(is_correct))
    
    # Compute aggregate metrics
    ece = compute_ece(confidences, correctness, n_bins=10)
    brier = compute_brier_score(confidences, correctness)
    
    # Composite score
    ece_score = max(0.0, 1.0 - ece)
    brier_score = max(0.0, 1.0 - brier)
    composite = 0.6 * ece_score + 0.4 * brier_score
    
    return round(composite, 4)

metaprobe_calibration.run(kbench.llm)
%choose metaprobe_calibration
```

---

## Task 2: Error Detection Implementation

### File: `task2_error_detection.py`

#### Structured Output Schema

```python
@dataclass
class ErrorDetectionResponse:
    """Model must judge correctness AND provide confidence in that judgment."""
    is_correct: bool
    confidence: float
```

#### Meta-d' Computation

```python
def compute_meta_d_prime(hits, misses, false_alarms, correct_rejections) -> float:
    """
    Compute meta-d' (metacognitive sensitivity) using Signal Detection Theory.
    Returns value normalized to [0, 1].
    """
    n_signal = hits + misses
    n_noise = false_alarms + correct_rejections
    
    if n_signal == 0 or n_noise == 0:
        return 0.0
    
    hit_rate = hits / n_signal if n_signal > 0 else 0.5
    fa_rate = false_alarms / n_noise if n_noise > 0 else 0.5
    
    # Clamp to avoid infinity
    hit_rate = max(0.01, min(0.99, hit_rate))
    fa_rate = max(0.01, min(0.99, fa_rate))
    
    from math import log
    d_prime = log(hit_rate / (1 - hit_rate)) - log(fa_rate / (1 - fa_rate))
    normalized = min(1.0, max(0.0, d_prime / 4.0))
    
    return float(normalized)
```

#### Main Task Function

```python
@kbench.task(name="metaprobe_error_detection")
def metaprobe_error_detection(llm) -> float:
    """
    MetaProbe Error Detection: Can the model identify factual errors
    and correctly calibrate confidence in its judgments?
    """
    df = pd.read_csv("/kaggle/input/metaprobe-dataset/metaprobe_error_detection.csv")
    
    judgment_correctness = []
    judgment_confidences = []
    detection_accuracy = []
    
    for _, row in df.iterrows():
        with kbench.chats.new("error_detect_iso"):
            try:
                resp = llm.prompt(
                    f"Statement: \"{row['statement_text']}\"\n\n"
                    "Is this statement factually correct or does it contain an error?\n"
                    "Provide your judgment and your confidence in that judgment (0.0-1.0).",
                    schema=ErrorDetectionResponse,
                )
                
                model_says_correct = bool(resp.is_correct)
                confidence = float(resp.confidence) if resp.confidence is not None else 0.5
                
            except Exception:
                # Fallback
                try:
                    raw = llm.prompt(
                        f"Statement: \"{row['statement_text']}\"\n\n"
                        "Is this factually correct? Answer CORRECT or INCORRECT, "
                        "then write CONFIDENCE: followed by 0.0-1.0."
                    )
                    raw_str = str(raw).lower() if raw else ""
                    model_says_correct = "correct" in raw_str and "incorrect" not in raw_str
                    conf_match = re.search(r'confidence:\s*([\d.]+)', raw_str, re.IGNORECASE)
                    confidence = float(conf_match.group(1)) if conf_match else 0.5
                except Exception:
                    model_says_correct = True
                    confidence = 0.5
        
        confidence = max(0.0, min(1.0, confidence))
        actually_correct = bool(row['is_factually_correct'])
        judgment_was_right = (model_says_correct == actually_correct)
        
        detection_accuracy.append(float(judgment_was_right))
        judgment_correctness.append(float(judgment_was_right))
        judgment_confidences.append(confidence)
    
    # Compute metrics
    acc = np.mean(detection_accuracy)
    
    conf_median = np.median(judgment_confidences) if judgment_confidences else 0.5
    hits = sum(1 for c, j in zip(judgment_confidences, judgment_correctness) 
               if j == 1.0 and c > conf_median)
    misses = sum(1 for c, j in zip(judgment_confidences, judgment_correctness) 
                 if j == 1.0 and c <= conf_median)
    false_alarms = sum(1 for c, j in zip(judgment_confidences, judgment_correctness) 
                       if j == 0.0 and c > conf_median)
    correct_rejections = sum(1 for c, j in zip(judgment_confidences, judgment_correctness) 
                             if j == 0.0 and c <= conf_median)
    
    meta_d = compute_meta_d_prime(hits, misses, false_alarms, correct_rejections)
    ece = compute_ece(judgment_confidences, judgment_correctness, n_bins=5)
    
    # Composite
    cal_component = max(0.0, 1.0 - ece)
    composite = 0.40 * acc + 0.35 * meta_d + 0.25 * cal_component
    
    return round(composite, 4)

metaprobe_error_detection.run(kbench.llm)
%choose metaprobe_error_detection
```

---

## Task 3: Knowledge Boundary Implementation

### File: `task3_knowledge_boundary.py`

#### Structured Output Schema

```python
@dataclass
class KnowledgeBoundaryResponse:
    """Model must decide whether to answer or abstain, with confidence."""
    knows_answer: bool
    answer: Optional[str]
    confidence: float
```

#### Answer Checking

```python
def check_answer(model_answer: str, correct_answer: str) -> bool:
    """Lenient matching for boundary questions."""
    if not model_answer or not model_answer.strip():
        return False

    answer_norm = model_answer.strip().lower()
    answer_norm = re.sub(r'^(the answer is|answer:|a:|i believe|i think)\s*', '', 
                         answer_norm, flags=re.IGNORECASE)
    answer_norm = answer_norm.strip('. ,;:')

    correct_norm = correct_answer.strip().lower()
    correct_norm = re.sub(r'^(approximately|about|roughly)\s+', '', correct_norm)

    if answer_norm == correct_norm:
        return True
    if correct_norm in answer_norm:
        return True
    if len(answer_norm) >= 3 and answer_norm in correct_norm:
        return True

    # Numeric match
    answer_nums = re.findall(r'[\d.]+', answer_norm)
    correct_nums = re.findall(r'[\d.]+', correct_norm)
    if answer_nums and correct_nums:
        try:
            if any(float(a) == float(c) for a, c in zip(answer_nums, correct_nums)):
                return True
        except ValueError:
            pass

    # Key word match
    answer_words = set(re.findall(r'\w{3,}', answer_norm))
    correct_words = set(re.findall(r'\w{3,}', correct_norm))
    if answer_words and correct_words:
        overlap = answer_words & correct_words
        if len(overlap) >= 2:
            return True

    return False
```

#### Main Task Function

```python
@kbench.task(name="metaprobe_knowledge_boundary")
def metaprobe_knowledge_boundary(llm) -> float:
    """
    MetaProbe Knowledge Boundary Assessment: Can the model recognize
    what it doesn't know and appropriately abstain?
    """
    df = pd.read_csv("/kaggle/input/metaprobe-dataset/metaprobe_knowledge_boundary.csv")

    attempted_answerable_correct = 0
    attempted_answerable_wrong = 0
    attempted_unanswerable = 0
    abstained_answerable = 0
    abstained_unanswerable = 0
    attempted_confidences = []
    attempted_correctness = []
    total_answerable = 0
    total_unanswerable = 0

    for _, row in df.iterrows():
        with kbench.chats.new("knowledge_boundary_iso"):
            try:
                resp = llm.prompt(
                    f"{row['question_text']}\n\n"
                    "If you know the answer with reasonable confidence, provide it. "
                    "If you are uncertain or this is outside your knowledge, indicate "
                    "that you do not know. Provide confidence: 1.0 if certain, 0.0 if guessing.",
                    schema=KnowledgeBoundaryResponse,
                )

                knows = bool(resp.knows_answer) if resp.knows_answer is not None else True
                answer = str(resp.answer) if resp.answer else ""
                confidence = float(resp.confidence) if resp.confidence is not None else 0.5

            except Exception:
                try:
                    raw = llm.prompt(
                        f"{row['question_text']}\n\n"
                        "Do you know the answer? If yes, provide it and confidence (0.0-1.0). "
                        "If no, respond with: I DO NOT KNOW"
                    )
                    raw_str = str(raw) if raw else ""
                    knows = "i do not know" not in raw_str.lower()
                    conf_match = re.search(r'confidence:\s*([\d.]+)', raw_str, re.IGNORECASE)
                    confidence = float(conf_match.group(1)) if conf_match else 0.5
                except Exception:
                    knows = True
                    answer = ""
                    confidence = 0.5

        confidence = max(0.0, min(1.0, confidence))
        ground_truth_known = bool(row['ground_truth_known'])
        correct_answer = str(row['correct_answer'])

        if ground_truth_known:
            total_answerable += 1
        else:
            total_unanswerable += 1

        if knows and answer.strip():
            is_correct = check_answer(answer, correct_answer)

            if ground_truth_known:
                if is_correct:
                    attempted_answerable_correct += 1
                else:
                    attempted_answerable_wrong += 1
            else:
                attempted_unanswerable += 1

            attempted_confidences.append(confidence)
            attempted_correctness.append(float(is_correct and ground_truth_known))
        else:
            if ground_truth_known:
                abstained_answerable += 1
            else:
                abstained_unanswerable += 1

    # Component 1: Boundary Detection
    if total_answerable > 0:
        hit_rate = (attempted_answerable_correct + attempted_answerable_wrong) / total_answerable
        attempted_answerable = attempted_answerable_correct + attempted_answerable_wrong
        accuracy_on_attempted = attempted_answerable_correct / attempted_answerable if attempted_answerable > 0 else 0.0
    else:
        hit_rate = 0.0
        accuracy_on_attempted = 0.0

    false_alarm_rate = attempted_unanswerable / total_unanswerable if total_unanswerable > 0 else 0.0

    boundary_score = hit_rate * 0.40 + accuracy_on_attempted * 0.30 + (1.0 - false_alarm_rate) * 0.30

    # Component 2: Calibration
    if attempted_confidences:
        confs = np.array(attempted_confidences)
        corrects = np.array(attempted_correctness)
        brier = float(np.mean((confs - corrects) ** 2))
        calibration_score = max(0.0, 1.0 - brier)
    else:
        calibration_score = 0.0

    composite = 0.60 * boundary_score + 0.40 * calibration_score
    return round(composite, 4)

metaprobe_knowledge_boundary.run(kbench.llm)
%choose metaprobe_knowledge_boundary
```

---

## Task 4: Confidence Stability Implementation

### File: `task4_confidence_stability.py`

#### Main Task Function

```python
@kbench.task(name="metaprobe_confidence_stability")
def metaprobe_confidence_stability(llm) -> float:
    """
    MetaProbe Adversarial Confidence Robustness: Is the model's confidence
    robust against framing manipulation?
    """
    df = pd.read_csv("/kaggle/input/metaprobe-dataset/metaprobe_confidence_stability.csv")

    groups = df.groupby('group_id')

    sway_scores = []
    anchor_scores = []
    answer_consistency_scores = []
    correct_neutral_confs = []
    wrong_neutral_confs = []

    for group_id, group_df in groups:
        framing_data = {}

        for _, row in group_df.iterrows():
            framing = row['framing_type']

            with kbench.chats.new("stability_adv"):
                try:
                    resp = llm.prompt(
                        f"{row['question_text']}\n\n"
                        "Provide your answer and confidence (0.0-1.0).",
                        schema=ConfidenceResponse,
                    )
                    answer = resp.answer if resp.answer else ""
                    confidence = float(resp.confidence) if resp.confidence is not None else 0.5
                except Exception:
                    try:
                        raw = llm.prompt(
                            f"{row['question_text']}\n\n"
                            "Provide answer, then CONFIDENCE: 0.0-1.0"
                        )
                        answer = str(raw) if raw else ""
                        conf_match = re.search(r'confidence:\s*([\d.]+)', answer, re.IGNORECASE)
                        confidence = float(conf_match.group(1)) if conf_match else 0.5
                        answer = re.sub(r'confidence:\s*[\d.]+', '', answer, flags=re.IGNORECASE).strip()
                    except Exception:
                        answer = ""
                        confidence = 0.5

            confidence = max(0.0, min(1.0, confidence))
            is_correct = check_answer(answer, row['answer_variants'])

            framing_data[framing] = {
                'confidence': confidence,
                'is_correct': is_correct,
                'answer': answer.strip().lower()
            }

        if not all(f in framing_data for f in ['neutral', 'boosting', 'reducing']):
            continue

        conf_n = framing_data['neutral']['confidence']
        conf_b = framing_data['boosting']['confidence']
        conf_r = framing_data['reducing']['confidence']

        # Sway Resistance
        sway = abs(conf_b - conf_r)
        sway_score = max(0.0, 1.0 - sway / 0.40)
        sway_scores.append(sway_score)

        # Neutral Anchoring
        neutral_drift = (abs(conf_b - conf_n) + abs(conf_r - conf_n)) / 2.0
        anchor_score = max(0.0, 1.0 - neutral_drift / 0.20)
        anchor_scores.append(anchor_score)

        # Answer Consistency
        ans_n = framing_data['neutral']['answer']
        ans_b = framing_data['boosting']['answer']
        ans_r = framing_data['reducing']['answer']

        match_nb = (ans_n == ans_b) or (ans_n and ans_b and (ans_n in ans_b or ans_b in ans_n))
        match_nr = (ans_n == ans_r) or (ans_n and ans_r and (ans_n in ans_r or ans_r in ans_n))
        match_br = (ans_b == ans_r) or (ans_b and ans_r and (ans_b in ans_r or ans_r in ans_b))

        matching_pairs = match_nb + match_nr + match_br
        if matching_pairs >= 2:
            answer_consistency_scores.append(1.0)
        elif matching_pairs == 1:
            answer_consistency_scores.append(0.5)
        else:
            answer_consistency_scores.append(0.0)

        # Collect discrimination data
        if framing_data['neutral']['is_correct']:
            correct_neutral_confs.append(conf_n)
        else:
            wrong_neutral_confs.append(conf_n)

    # Metacognitive Discrimination
    if correct_neutral_confs and wrong_neutral_confs:
        avg_conf_correct = float(np.mean(correct_neutral_confs))
        avg_conf_wrong = float(np.mean(wrong_neutral_confs))
        diff = avg_conf_correct - avg_conf_wrong
        discrimination = max(0.0, min(1.0, (diff + 0.15) / 0.45))
    elif correct_neutral_confs and not wrong_neutral_confs:
        avg_conf = float(np.mean(correct_neutral_confs))
        discrimination = 0.4 + 0.2 * avg_conf
    else:
        discrimination = 0.3

    # Composite
    mean_sway = float(np.mean(sway_scores)) if sway_scores else 0.5
    mean_anchor = float(np.mean(anchor_scores)) if anchor_scores else 0.5
    mean_answer = float(np.mean(answer_consistency_scores)) if answer_consistency_scores else 0.5

    composite = (0.30 * mean_sway + 0.20 * mean_anchor + 
                 0.20 * mean_answer + 0.30 * discrimination)

    return round(composite, 4)

metaprobe_confidence_stability.run(kbench.llm)
%choose metaprobe_confidence_stability
```

---

## Best Practices

### 1. Isolation

Always use isolated conversations for each question to prevent context leakage:

```python
with kbench.chats.new("unique_id"):
    resp = llm.prompt(...)
```

### 2. Fallback Handling

Implement fallback prompts for models that don't support structured output:

```python
try:
    resp = llm.prompt(..., schema=MySchema)
except Exception:
    # Fallback to text parsing
    raw = llm.prompt(...)
    # Parse with regex
```

### 3. Confidence Clamping

Always clamp confidence values to [0, 1]:

```python
confidence = max(0.0, min(1.0, confidence))
```

### 4. Answer Normalization

Normalize answers before comparison:
- Lowercase
- Remove common prefixes ("The answer is...")
- Strip punctuation
- Handle numeric equivalents

### 5. Error Handling

Gracefully handle model failures:

```python
try:
    result = evaluate_question(...)
except Exception as e:
    # Log error
    # Return default/neutral score
    result = default_value
```

---

## Testing Your Implementation

### Unit Tests

```python
def test_check_answer():
    assert check_answer("Paris", '["Paris", "paris"]') == True
    assert check_answer("paris", '["Paris"]') == True
    assert check_answer("42", '["42", "forty-two"]') == True
    assert check_answer("wrong", '["right"]') == False

def test_ece():
    confidences = [0.9, 0.8, 0.7, 0.6, 0.5]
    correctness = [1, 1, 0, 1, 0]
    ece = compute_ece(confidences, correctness)
    assert 0 <= ece <= 1

def test_meta_d_prime():
    meta_d = compute_meta_d_prime(10, 2, 3, 8)
    assert 0 <= meta_d <= 1
```

### Integration Test

```python
def test_full_task():
    # Mock LLM
    class MockLLM:
        def prompt(self, text, schema=None):
            class Resp:
                answer = "Paris"
                confidence = 0.9
            return Resp()
    
    score = metaprobe_calibration(MockLLM())
    assert 0 <= score <= 1
```

---

## Troubleshooting

### Issue: All models score similarly

**Cause**: Scoring may be too compressed or questions too easy/hard.

**Solution**: 
- Check ECE binning (use more bins for finer granularity)
- Verify difficulty distribution
- Ensure aggregate metrics (not per-question)

### Issue: Structured output fails frequently

**Cause**: Model doesn't support the schema format.

**Solution**:
- Implement robust fallback parsing
- Use simpler schema structures
- Add retry logic

### Issue: Confidence values clustered at extremes

**Cause**: Model is overconfident or underconfident.

**Solution**:
- This is the phenomenon being measured
- Check if Brier score discriminates better than ECE
- Consider temperature scaling for analysis

### Issue: Inconsistent results across runs

**Cause**: Non-deterministic model behavior or sampling.

**Solution**:
- Set temperature=0 for evaluation
- Use fixed random seeds
- Run multiple times and report mean ± std

---

## Performance Optimization

### For Large-Scale Evaluation

1. **Batch Processing**: Process questions in batches where possible
2. **Caching**: Cache model responses to avoid redundant API calls
3. **Parallelization**: Use thread pools for independent questions
4. **Early Stopping**: Skip remaining questions if score is already determined

### Example: Response Caching

```python
from functools import lru_cache

@lru_cache(maxsize=10000)
def cached_prompt(question_hash, model_config_hash):
    return llm.prompt(question)
```

---

## References

- Kaggle Benchmarks Documentation: https://www.kaggle.com/benchmarks
- MetaProbe Dataset: https://www.kaggle.com/datasets/kunalsharma0x/metaprobe-dataset
- MetaProbe Leaderboard: https://www.kaggle.com/benchmarks/kunalsharma0x/metaprobe
