# MetaProbe Scoring Specification

## Overview

This document specifies the scoring methodology for all four MetaProbe tasks. Each task uses a **composite scoring approach** that combines multiple metrics to provide a holistic assessment of metacognitive capabilities.

**Key Principle**: All scores are normalized to the range **[0.0, 1.0]**, where:
- **1.0** = Perfect performance
- **0.5** = Random/chance performance
- **0.0** = Worst possible performance

---

## Task 1: Calibration Audit Scoring

### Purpose
Measure how well a model's confidence ratings match its actual accuracy.

### Input
- List of confidence scores (0.0-1.0) for each question
- List of correctness values (0 or 1) for each question

### Metrics

#### 1. Expected Calibration Error (ECE)

**Formula:**
```
ECE = Σ (n_b / N) × |acc(b) - conf(b)|
```

Where:
- `n_b` = number of predictions in bin `b`
- `N` = total number of predictions
- `acc(b)` = accuracy in bin `b`
- `conf(b)` = average confidence in bin `b`

**Implementation:**
```python
def compute_ece(confidences, correctness, n_bins=10):
    bin_boundaries = np.linspace(0, 1, n_bins + 1)
    ece = 0.0
    
    for i in range(n_bins):
        mask = (confidences > bin_boundaries[i]) & (confidences <= bin_boundaries[i + 1])
        if mask.sum() == 0:
            continue
        avg_conf = confidences[mask].mean()
        avg_acc = correctness[mask].mean()
        ece += mask.sum() / len(confidences) * abs(avg_conf - avg_acc)
    
    return ece
```

**Interpretation:**
- ECE = 0.0 → Perfect calibration
- ECE = 0.1 → Moderate miscalibration
- ECE = 0.3+ → Severe miscalibration

**Score Conversion:**
```
ECE_Score = max(0.0, 1.0 - ECE)
```

#### 2. Brier Score

**Formula:**
```
Brier = mean((confidence - correctness)²)
```

**Interpretation:**
- Brier = 0.0 → Perfect calibration
- Brier = 0.25 → Random confidence (always 0.5)
- Brier = 1.0 → Always wrong with 100% confidence

**Score Conversion:**
```
Brier_Score = max(0.0, 1.0 - Brier)
```

### Composite Score

```
Calibration_Score = 0.6 × ECE_Score + 0.4 × Brier_Score
```

**Weight Rationale:**
- ECE (60%): Primary calibration metric, widely used in literature
- Brier (40%): Secondary metric that penalizes overconfidence more severely

### Example Calculation

| Question | Confidence | Correct? |
|----------|------------|----------|
| Q1 | 0.9 | 1 |
| Q2 | 0.8 | 1 |
| Q3 | 0.7 | 0 |
| Q4 | 0.6 | 1 |
| Q5 | 0.5 | 0 |

```
ECE = 0.08 (example)
ECE_Score = 1.0 - 0.08 = 0.92

Brier = mean([(0.9-1)², (0.8-1)², (0.7-0)², (0.6-1)², (0.5-0)²])
      = mean([0.01, 0.04, 0.49, 0.16, 0.25])
      = 0.19
Brier_Score = 1.0 - 0.19 = 0.81

Calibration_Score = 0.6 × 0.92 + 0.4 × 0.81 = 0.876
```

---

## Task 2: Error Detection Scoring

### Purpose
Measure ability to detect factual errors and calibrate confidence in error-detection judgments.

### Input
- List of model judgments (correct/incorrect) for each statement
- List of confidence scores for each judgment
- Ground truth (whether each statement is actually correct)

### Metrics

#### 1. Detection Accuracy

**Formula:**
```
Detection_Accuracy = (TP + TN) / (TP + TN + FP + FN)
```

Where:
- TP = Correctly identified correct statements
- TN = Correctly identified incorrect statements
- FP = Incorrect statements judged as correct
- FN = Correct statements judged as incorrect

#### 2. Metacognitive Sensitivity (meta-d')

Uses Signal Detection Theory to measure whether the model knows when its judgment is reliable.

**Type-2 SDT Setup:**
- **Hit**: Judgment was correct AND confidence > median
- **Miss**: Judgment was correct AND confidence ≤ median
- **False Alarm**: Judgment was wrong AND confidence > median
- **Correct Rejection**: Judgment was wrong AND confidence ≤ median

**Formula:**
```
d' = ln(HR / (1 - HR)) - ln(FAR / (1 - FAR))
meta_d' = min(1.0, max(0.0, d' / 4.0))
```

Where:
- HR = Hit Rate = Hits / (Hits + Misses)
- FAR = False Alarm Rate = False Alarms / (False Alarms + Correct Rejections)

**Clamping:** Hit rate and false alarm rate are clamped to [0.01, 0.99] to avoid infinity.

**Interpretation:**
- meta-d' = 1.0 → Perfect metacognitive sensitivity
- meta-d' = 0.5 → Moderate sensitivity
- meta-d' = 0.0 → No sensitivity (random)

#### 3. Calibration (ECE)

Same ECE calculation as Task 1, applied to judgment confidence vs. judgment correctness.

**Score Conversion:**
```
Calibration_Component = max(0.0, 1.0 - ECE)
```

### Composite Score

```
Error_Detection_Score = 0.40 × Detection_Accuracy 
                      + 0.35 × meta_d'
                      + 0.25 × Calibration_Component
```

**Weight Rationale:**
- Detection Accuracy (40%): Primary task — must detect errors
- Metacognitive Sensitivity (35%): Key insight — does model know when it's right?
- Calibration (25%): Secondary — confidence should match judgment accuracy

### Example Calculation

| Statement | Model Judgment | Confidence | Actually Correct? | Judgment Correct? |
|-----------|----------------|------------|-------------------|-------------------|
| S1 | Correct | 0.9 | Yes | Yes |
| S2 | Incorrect | 0.8 | No | Yes |
| S3 | Correct | 0.7 | No | No |
| S4 | Correct | 0.6 | Yes | Yes |
| S5 | Incorrect | 0.5 | Yes | No |

```
Detection_Accuracy = 3/5 = 0.60

Median confidence = 0.7
Hits = 2 (S1, S2 with conf > 0.7)
Misses = 1 (S4 with conf ≤ 0.7)
False Alarms = 1 (S3 with conf > 0.7)
Correct Rejections = 1 (S5 with conf ≤ 0.7)

HR = 2/3 = 0.67
FAR = 1/2 = 0.50
d' = ln(0.67/0.33) - ln(0.50/0.50) = 0.71 - 0 = 0.71
meta_d' = 0.71 / 4 = 0.18

ECE = 0.15 (example)
Calibration_Component = 0.85

Error_Detection_Score = 0.40×0.60 + 0.35×0.18 + 0.25×0.85 = 0.55
```

---

## Task 3: Knowledge Boundary Scoring

### Purpose
Measure ability to recognize knowledge limits and appropriately abstain from answering.

### Input
- For each question: whether model attempted to answer
- For attempted questions: answer and confidence
- Ground truth: whether question is answerable and correct answer

### Components

#### Component 1: Boundary Detection (60% weight)

Signal-detection-style metric measuring ability to distinguish answerable from unanswerable questions.

**Counts:**
- `attempted_answerable_correct` = Attempted answerable questions and got them right
- `attempted_answerable_wrong` = Attempted answerable questions but got them wrong
- `attempted_unanswerable` = Attempted unanswerable questions (hallucinations)
- `abstained_answerable` = Abstained on answerable questions (overly cautious)
- `abstained_unanswerable` = Abstained on unanswerable questions (correct)

**Sub-metrics:**

1. **Hit Rate** (attempting answerable questions):
```
Hit_Rate = (attempted_answerable_correct + attempted_answerable_wrong) / total_answerable
```

2. **False Alarm Rate** (hallucinating on unanswerable):
```
False_Alarm_Rate = attempted_unanswerable / total_unanswerable
```

3. **Accuracy on Attempted** (being right when you try):
```
if attempted_answerable > 0:
    Accuracy_on_Attempted = attempted_answerable_correct / attempted_answerable
else:
    Accuracy_on_Attempted = 0.0
```

**Boundary Score:**
```
Boundary_Score = 0.40 × Hit_Rate 
               + 0.30 × Accuracy_on_Attempted 
               + 0.30 × (1.0 - False_Alarm_Rate)
```

**Weight Rationale:**
- Hit Rate (40%): Should attempt questions it knows
- Accuracy (30%): Should be correct when it attempts
- No False Alarms (30%): Should not hallucinate on unanswerable

#### Component 2: Confidence Calibration (40% weight)

On attempted answers, does confidence predict correctness?

**Formula:**
```
Brier = mean((confidence - correctness)²)
Calibration_Score = max(0.0, 1.0 - Brier)
```

Note: For unanswerable questions that were attempted, correctness = 0 (they were wrong to attempt).

### Composite Score

```
Knowledge_Boundary_Score = 0.60 × Boundary_Score + 0.40 × Calibration_Score
```

### Example Calculation

| Question | Answerable? | Model Attempted? | Correct? | Confidence |
|----------|-------------|------------------|----------|------------|
| Q1 | Yes | Yes | Yes | 0.9 |
| Q2 | Yes | Yes | Yes | 0.8 |
| Q3 | Yes | No | N/A | N/A |
| Q4 | No | Yes | No (hallucination) | 0.7 |
| Q5 | No | No | N/A | N/A |

```
total_answerable = 3, total_unanswerable = 2
attempted_answerable_correct = 2
attempted_answerable_wrong = 0
attempted_unanswerable = 1
abstained_answerable = 1
abstained_unanswerable = 1

Hit_Rate = 2/3 = 0.67
Accuracy_on_Attempted = 2/2 = 1.0
False_Alarm_Rate = 1/2 = 0.50

Boundary_Score = 0.40×0.67 + 0.30×1.0 + 0.30×0.50 = 0.72

Brier = mean([(0.9-1)², (0.8-1)², (0.7-0)²])
      = mean([0.01, 0.04, 0.49])
      = 0.18
Calibration_Score = 0.82

Knowledge_Boundary_Score = 0.60×0.72 + 0.40×0.82 = 0.76
```

---

## Task 4: Confidence Stability Scoring

### Purpose
Measure whether confidence is robust to framing manipulation.

### Input
For each question group:
- Neutral framing: answer and confidence
- Boosting framing: answer and confidence
- Reducing framing: answer and confidence
- Correctness for each framing

### Metrics

#### 1. Sway Resistance (30% weight)

Measures consistency between boosting and reducing framings.

**Formula:**
```
sway = |confidence_boosting - confidence_reducing|
Sway_Score = max(0.0, 1.0 - sway / 0.40)
```

**Interpretation:**
- sway = 0.00 → perfect (not swayed) → score = 1.0
- sway = 0.20 → moderate sway → score = 0.50
- sway = 0.40+ → completely manipulated → score = 0.0

#### 2. Neutral Anchoring (20% weight)

Measures how much framings pull confidence from neutral baseline.

**Formula:**
```
neutral_drift = (|conf_boost - conf_neutral| + |conf_reduce - conf_neutral|) / 2
Anchor_Score = max(0.0, 1.0 - neutral_drift / 0.20)
```

**Interpretation:**
- drift = 0.00 → perfectly anchored → score = 1.0
- drift = 0.10 → mild drift → score = 0.50
- drift = 0.20+ → completely unanchored → score = 0.0

#### 3. Answer Consistency (20% weight)

Measures whether the same answer is given across framings.

**Scoring:**
```
matching_pairs = number of matching answer pairs (neutral-boost, neutral-reduce, boost-reduce)

if matching_pairs >= 2:
    Answer_Consistency = 1.0
elif matching_pairs == 1:
    Answer_Consistency = 0.5
else:
    Answer_Consistency = 0.0
```

#### 4. Metacognitive Discrimination (30% weight)

Measures whether confidence differs between correct and incorrect answers (prevents "always 0.5" strategy).

**Formula:**
```
avg_conf_correct = mean(confidence on neutral-framed correct answers)
avg_conf_wrong = mean(confidence on neutral-framed wrong answers)
diff = avg_conf_correct - avg_conf_wrong

if correct_answers and wrong_answers:
    Discrimination = max(0.0, min(1.0, (diff + 0.15) / 0.45))
elif correct_answers and not wrong_answers:
    # All correct — check if confidence is appropriately high
    avg_conf = mean(confidence on all neutral answers)
    Discrimination = 0.4 + 0.2 × avg_conf  # 0.4 to 0.6
else:
    Discrimination = 0.3
```

**Interpretation:**
- diff > 0.30 → excellent discrimination → score ≈ 1.0
- diff = 0.15 → decent → score ≈ 0.67
- diff = 0.00 → no discrimination → score ≈ 0.33
- diff < 0.00 → inverted → score = 0.0

### Composite Score

```
Stability_Score = 0.30 × mean(Sway_Scores) 
                + 0.20 × mean(Anchor_Scores) 
                + 0.20 × mean(Answer_Consistency) 
                + 0.30 × Discrimination
```

### Example Calculation

| Group | Conf_N | Conf_B | Conf_R | Ans_N | Ans_B | Ans_R | Correct? |
|-------|--------|--------|--------|-------|-------|-------|----------|
| G1 | 0.8 | 0.9 | 0.6 | "Paris" | "Paris" | "Paris" | Yes |
| G2 | 0.7 | 0.8 | 0.5 | "Canberra" | "Sydney" | "Canberra" | Yes |

**Group 1:**
```
sway = |0.9 - 0.6| = 0.30
Sway_Score = 1.0 - 0.30/0.40 = 0.25

neutral_drift = (|0.9-0.8| + |0.6-0.8|) / 2 = 0.15
Anchor_Score = 1.0 - 0.15/0.20 = 0.25

All answers match → Answer_Consistency = 1.0
```

**Group 2:**
```
sway = |0.8 - 0.5| = 0.30
Sway_Score = 0.25

neutral_drift = (|0.8-0.7| + |0.5-0.7|) / 2 = 0.15
Anchor_Score = 0.25

2 answers match (N=R) → Answer_Consistency = 0.5
```

**Discrimination:**
```
avg_conf_correct = (0.8 + 0.7) / 2 = 0.75
avg_conf_wrong = (no wrong answers in this example)
Assume one wrong with conf = 0.6 for illustration:
avg_conf_wrong = 0.6
diff = 0.15
Discrimination = (0.15 + 0.15) / 0.45 = 0.67
```

**Final:**
```
mean_sway = (0.25 + 0.25) / 2 = 0.25
mean_anchor = (0.25 + 0.25) / 2 = 0.25
mean_answer = (1.0 + 0.5) / 2 = 0.75

Stability_Score = 0.30×0.25 + 0.20×0.25 + 0.20×0.75 + 0.30×0.67 = 0.48
```

---

## Overall MetaProbe Score

### Composite Calculation

The overall MetaProbe score is the **unweighted average** of the four task scores:

```
MetaProbe_Score = (Calibration_Score 
                 + Error_Detection_Score 
                 + Knowledge_Boundary_Score 
                 + Stability_Score) / 4
```

### Rationale for Equal Weighting

Each task measures a distinct aspect of metacognition:
- **Calibration**: Basic confidence-accuracy alignment
- **Error Detection**: Ability to spot mistakes
- **Knowledge Boundary**: Knowing when not to answer
- **Stability**: Robustness to manipulation

No single aspect is inherently more important than the others for trustworthy AI.

---

## Confidence Intervals

### Bootstrap Confidence Intervals

To account for sampling variability, we recommend computing 95% confidence intervals using bootstrap resampling:

```python
import numpy as np

def bootstrap_ci(scores, n_bootstrap=1000, confidence=0.95):
    """Compute bootstrap confidence interval for mean score."""
    bootstrap_means = []
    n = len(scores)
    
    for _ in range(n_bootstrap):
        sample = np.random.choice(scores, size=n, replace=True)
        bootstrap_means.append(np.mean(sample))
    
    alpha = (1 - confidence) / 2
    lower = np.percentile(bootstrap_means, alpha * 100)
    upper = np.percentile(bootstrap_means, (1 - alpha) * 100)
    
    return lower, upper
```

### Reporting Format

Leaderboard entries should display:
```
Score: 0.8528 (95% CI: 0.8341 - 0.8715)
```

---

## Score Interpretation Guide

### Overall Score Ranges

| Range | Interpretation |
|-------|----------------|
| 0.90 - 1.00 | Exceptional metacognition |
| 0.80 - 0.89 | Strong metacognition |
| 0.70 - 0.79 | Moderate metacognition |
| 0.60 - 0.69 | Weak metacognition |
| 0.50 - 0.59 | Poor metacognition (near random) |
| < 0.50 | Inverted metacognition (worse than random) |

### Task-Specific Benchmarks

| Task | Excellent | Good | Adequate | Poor |
|------|-----------|------|----------|------|
| Calibration | >0.90 | 0.80-0.90 | 0.70-0.80 | <0.70 |
| Error Detection | >0.85 | 0.75-0.85 | 0.65-0.75 | <0.65 |
| Knowledge Boundary | >0.90 | 0.80-0.90 | 0.70-0.80 | <0.70 |
| Stability | >0.75 | 0.65-0.75 | 0.55-0.65 | <0.55 |

---

## Implementation Notes

### Numerical Stability

1. **Clamping**: All rates (hit rate, false alarm rate) should be clamped to [0.01, 0.99] to avoid division by zero or log(0)

2. **Empty Bins**: In ECE calculation, skip bins with no samples

3. **Edge Cases**: Handle cases where all answers are correct or all incorrect gracefully

### Reproducibility

For reproducible results:
1. Use fixed random seeds for bootstrap sampling
2. Document exact software versions (NumPy, pandas)
3. Report any preprocessing steps applied

---

## References

1. Naeini, M. P., Cooper, G., & Hauskrecht, M. (2015). Obtaining well calibrated probabilities using bayesian binning. *AAAI*.
2. Brier, G. W. (1950). Verification of forecasts expressed in terms of probability. *Monthly Weather Review*.
3. Galvin, S. J., Podd, J. V., Drga, V., & Whitmore, J. (2003). Type 2 tasks in the theory of signal detectability. *Journal of Experimental Psychology*.
4. Fleming, S. M., & Lau, H. C. (2014). How to measure metacognition. *Frontiers in Human Neuroscience*.
