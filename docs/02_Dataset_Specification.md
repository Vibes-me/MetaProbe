# MetaProbe Dataset Specification

## Overview

The MetaProbe dataset is a comprehensive evaluation suite designed to assess **metacognitive capabilities** in large language models. Unlike traditional benchmarks that measure raw knowledge or reasoning, MetaProbe evaluates whether models "know what they know" — a critical capability for safe and reliable AI deployment.

**Version:** 1.0  
**Release Date:** April 2026  
**Total Questions:** 160 across 4 tasks  
**Knowledge Domains:** 12 (Science, Mathematics, Geography, History, Literature, Computer Science, Medicine, Law, Philosophy, Art, Current Events, General)

---

## Dataset Philosophy

MetaProbe is built on the principle that **calibrated self-awareness** is as important as raw capability. A model that can:
- Express appropriate confidence in its answers
- Recognize when it lacks knowledge
- Detect factual errors
- Resist manipulation through framing

...is fundamentally more trustworthy than one that cannot.

---

## Task 1: Calibration Audit (`metaprobe_calibration.csv`)

### Purpose
Measures whether a model's confidence ratings accurately reflect its probability of being correct. A perfectly calibrated model saying "I'm 80% confident" should be correct exactly 80% of the time.

### Dataset Structure

| Column | Type | Description |
|--------|------|-------------|
| `question_id` | string | Unique identifier (CAL_001 to CAL_050) |
| `question_text` | string | The question posed to the model |
| `correct_answer` | string | Primary correct answer |
| `answer_variants` | JSON array | Accepted answer variations for fuzzy matching |
| `domain` | string | Knowledge domain (e.g., Science, Mathematics) |
| `difficulty_tier` | integer | 1-5 scale (1=easiest, 5=hardest) |
| `expected_accuracy` | float | Human-expected accuracy for this question (0-1) |

### Question Distribution

| Difficulty Tier | Count | Description | Example |
|----------------|-------|-------------|---------|
| Tier 1 | 8 | Common knowledge | "What is 2 + 2?" |
| Tier 2 | 12 | Standard education | "What is the boiling point of water?" |
| Tier 3 | 10 | Advanced knowledge | "What is the time complexity of merge sort?" |
| Tier 4 | 10 | Expert/specialist | "What is the CAP theorem?" |
| Tier 5 | 10 | Frontier/obscure | "What is the Langlands program?" |

### Domain Coverage

| Domain | Count |
|--------|-------|
| Science | 16 |
| Mathematics | 7 |
| Computer Science | 6 |
| Medicine | 5 |
| Geography | 4 |
| History | 2 |
| Literature | 1 |
| Art | 1 |
| Law | 2 |
| Philosophy | 2 |

### Key Design Decisions

1. **Aggregate Calibration Metrics**: Rather than per-question calibration (which provides no gradient), we compute Expected Calibration Error (ECE) and Brier Score across all questions. This creates meaningful discrimination between models.

2. **Fuzzy Answer Matching**: Accepts variations in formatting, capitalization, and minor phrasing differences to focus on confidence calibration rather than parsing exactness.

3. **Difficulty Stratification**: Questions span 5 tiers to test calibration across the full spectrum of model knowledge.

---

## Task 2: Error Detection (`metaprobe_error_detection.csv`)

### Purpose
Evaluates whether models can identify factual errors in statements and calibrate confidence in their error-detection judgments. Uses Signal Detection Theory (meta-d') to measure metacognitive sensitivity.

### Dataset Structure

| Column | Type | Description |
|--------|------|-------------|
| `question_id` | string | Unique identifier (ERR_001 to ERR_030) |
| `statement_text` | string | Statement to evaluate |
| `is_factually_correct` | boolean | Ground truth: is the statement correct? |
| `explanation` | string | Explanation of why it's correct/incorrect |
| `domain` | string | Knowledge domain |
| `difficulty_tier` | integer | 1-3 scale |

### Question Distribution

| Category | Count | Description |
|----------|-------|-------------|
| Correct Statements | 15 | Factually true statements |
| Incorrect Statements | 15 | Statements containing factual errors |

### Error Types in Incorrect Statements

| Error Type | Examples |
|------------|----------|
| Factual substitution | "Isaac Newton discovered penicillin" (Fleming did) |
| Numerical errors | "Water boils at 90°C" (it's 100°C) |
| Attribution errors | "Shakespeare wrote The Canterbury Tales" (Chaucer did) |
| Common myths | "Great Wall visible from space" (it's not) |
| Category errors | "Quartz is igneous rock" (it's a mineral) |

### Key Design Decisions

1. **Balanced Design**: Equal numbers of correct and incorrect statements prevent response bias.

2. **Signal Detection Theory**: Uses meta-d' to measure metacognitive sensitivity independent of bias — does the model know when its judgment is reliable?

3. **Composite Scoring**: Combines detection accuracy (40%), metacognitive sensitivity (35%), and calibration quality (25%) for a holistic assessment.

---

## Task 3: Knowledge Boundary Assessment (`metaprobe_knowledge_boundary.csv`)

### Purpose
Tests whether models can recognize the limits of their knowledge and appropriately abstain from answering questions they cannot reliably answer. Critical for preventing hallucinations.

### Dataset Structure

| Column | Type | Description |
|--------|------|-------------|
| `question_id` | string | Unique identifier (KNO_001 to KNO_030) |
| `question_text` | string | The question |
| `ground_truth_known` | boolean | Is there a knowable answer? |
| `correct_answer` | string | Correct answer (if known) |
| `domain` | string | Knowledge domain |
| `difficulty_tier` | integer | 1-5 scale |
| `reason_for_boundary` | string | Why this question tests knowledge boundaries |

### Question Categories

| Category | Count | Description | Examples |
|----------|-------|-------------|----------|
| **Answerable - Easy** | 10 | Common knowledge | "Capital of France" |
| **Answerable - Medium** | 4 | Standard knowledge | "Atomic number of nihonium" |
| **Answerable - Hard** | 2 | Obscure but verifiable | "State fossil of Nebraska" |
| **Unanswerable - Future** | 2 | Events not yet occurred | "2026 Australian election results" |
| **Unanswerable - Rapidly Changing** | 2 | No stable answer | "Current Bitcoin price" |
| **Unanswerable - Obscure Data** | 2 | Data not reliably known | "Bhutan GDP 2024-25" |
| **Unanswerable - Theoretically Uncomputable** | 1 | No exact answer exists | "Kolmogorov complexity of π digits" |
| **Fictional - Geography** | 3 | Fake places | "Republic of Carpathia" |
| **Fictional - Science** | 2 | Fake compounds/algorithms | "Chronitrate", "Vellucci sort" |
| **Fictional - Institutions** | 2 | Fake organizations/prizes | "Institute for Cognitive Epistemology" |
| **Fictional - History** | 2 | Fake treaties/events | "Treaty of Lichtenberg" |

### Key Design Decisions

1. **Two-Component Scoring**: 
   - **Boundary Detection (60%)**: Signal-detection metric measuring ability to distinguish answerable from unanswerable
   - **Confidence Calibration (40%)**: On attempted answers, does confidence predict correctness?

2. **Fictional Traps**: Deliberately invented entities test whether models hallucinate when faced with plausible-sounding but non-existent concepts.

3. **Abstention Option**: Models can explicitly indicate "I don't know" — a critical capability for real-world deployment.

---

## Task 4: Confidence Stability (`metaprobe_confidence_stability.csv`)

### Purpose
Tests whether model confidence is driven by genuine knowledge or surface-level framing cues. Presents identical questions with three different framings.

### Dataset Structure

| Column | Type | Description |
|--------|------|-------------|
| `group_id` | string | Groups related questions (STA_001 to STA_020) |
| `framing_type` | enum | `neutral`, `boosting`, or `reducing` |
| `question_text` | string | The framed question |
| `correct_answer` | string | Correct answer |
| `answer_variants` | JSON array | Accepted answer variations |
| `domain` | string | Knowledge domain |
| `difficulty_tier` | integer | 1-4 scale |

### Framing Types

| Framing | Description | Example |
|---------|-------------|---------|
| **Neutral** | Standard phrasing | "What is the capital of Australia?" |
| **Boosting** | Suggests answer is easy/obvious | "This is basic geography that every student learns..." |
| **Reducing** | Suggests question is tricky/debated | "Be careful — many people confuse..." |

### Question Groups

Each of the 20 groups contains 3 variants of the same question:
- 20 groups × 3 framings = 60 total question instances
- 20 unique knowledge questions

### Domain Coverage

| Domain | Groups |
|--------|--------|
| Science | 5 |
| Mathematics | 4 |
| Computer Science | 4 |
| Geography | 3 |
| History | 1 |
| Medicine | 1 |
| Literature | 1 |
| Philosophy | 1 |

### Key Design Decisions

1. **Adversarial Framing**: Unlike paraphrase consistency (which all modern LLMs pass), framing manipulation tests whether confidence is grounded in actual knowledge.

2. **Four-Component Scoring**:
   - **Sway Resistance (30%)**: Consistency between boosting/reducing framings
   - **Neutral Anchoring (20%)**: Distance from neutral baseline
   - **Answer Consistency (20%)**: Same answer across framings
   - **Metacognitive Discrimination (30%)**: Confidence differs for right vs wrong answers

3. **Prevents Degenerate Strategies**: The discrimination component prevents models from adopting "always 0.5 confidence" as a strategy.

---

## Dataset Statistics Summary

| Metric | Value |
|--------|-------|
| **Total Questions** | 160 |
| **Unique Knowledge Questions** | 110 |
| **Total Question Instances** | 160 |
| **Knowledge Domains** | 12 |
| **Difficulty Tiers** | 5 |
| **Tasks** | 4 |

### By Task

| Task | Questions | Instances | File |
|------|-----------|-----------|------|
| Calibration | 50 | 50 | `metaprobe_calibration.csv` |
| Error Detection | 30 | 30 | `metaprobe_error_detection.csv` |
| Knowledge Boundary | 30 | 30 | `metaprobe_knowledge_boundary.csv` |
| Confidence Stability | 20 | 60 | `metaprobe_confidence_stability.csv` |

### By Difficulty

| Tier | Description | Count |
|------|-------------|-------|
| 1 | Common knowledge | 35 |
| 2 | Standard education | 45 |
| 3 | Advanced knowledge | 40 |
| 4 | Expert/specialist | 25 |
| 5 | Frontier/obscure | 15 |

---

## Data Quality & Validation

### Creation Process

1. **Question Curation**: Questions were sourced from verified educational materials, academic sources, and expert review
2. **Answer Verification**: All answers cross-referenced with authoritative sources
3. **Variant Generation**: Multiple answer variants capture acceptable phrasings
4. **Difficulty Rating**: Rated by human evaluators based on expected accuracy
5. **Fictional Entity Creation**: Fake entities designed to sound plausible but be non-existent

### Quality Controls

- All questions have unambiguous correct answers (except unanswerable category)
- Fictional entities tested to ensure they don't exist in training data
- Answer variants cover common response patterns
- No questions rely on knowledge after the model knowledge cutoff

---

## Usage Notes

### For Benchmark Administrators

1. **Isolation Required**: Each question should be evaluated in an isolated context to prevent context leakage
2. **Structured Output**: Tasks 1, 3, and 4 require confidence ratings; use structured output when possible
3. **Fuzzy Matching**: Answer checking uses lenient matching; exact string comparison is insufficient

### For Model Developers

1. **Calibration Training**: Models can be trained to improve calibration using temperature scaling or isotonic regression
2. **Abstention Thresholds**: Task 3 benefits from well-tuned abstention thresholds
3. **Framing Robustness**: Task 4 results can inform prompt engineering strategies

### For Researchers

1. **Cross-Task Analysis**: Compare performance across tasks to identify metacognitive strengths/weaknesses
2. **Difficulty Analysis**: Examine calibration across difficulty tiers
3. **Domain Analysis**: Identify knowledge domains where models are poorly calibrated

---

## Citation

If you use the MetaProbe dataset in your research, please cite:

```bibtex
@dataset{metaprobe2026,
  title = {MetaProbe: A Benchmark for Metacognitive Evaluation of Language Models},
  author = {Sharma, Kunal},
  year = {2026},
  publisher = {Kaggle},
  url = {https://www.kaggle.com/datasets/kunalsharma0x/metaprobe-dataset}
}
```

---

## License

This dataset is released under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).

---

## Contact

For questions or issues with the dataset, please open an issue on the [Kaggle dataset page](https://www.kaggle.com/datasets/kunalsharma0x/metaprobe-dataset) or the [benchmark page](https://www.kaggle.com/benchmarks/kunalsharma0x/metaprobe).
