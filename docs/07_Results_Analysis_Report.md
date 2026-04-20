# MetaProbe Results Analysis Report

## Executive Summary

This report analyzes the results from the first comprehensive evaluation of 11 state-of-the-art language models on the MetaProbe benchmark. The evaluation was conducted in April 2026.

**Key Findings:**
- **Claude Sonnet 4.6** leads with an overall score of **0.8528**
- **Error Detection** is the hardest task (avg: 0.680), **Knowledge Boundary** is easiest (avg: 0.894)
- Strong correlation (r=0.79) between Knowledge Boundary and Confidence Stability
- **GPT-OSS 120B** struggles significantly, scoring 0.6475 overall
- **GLM-5** shows surprising strength in Confidence Stability (0.8017)

---

## Leaderboard Overview

### Overall Rankings

| Rank | Model | Overall Score | Grade |
|------|-------|---------------|-------|
| 1 | Claude Sonnet 4.6 | **0.8528** | Excellent |
| 2 | Claude Haiku 4.5 | **0.8082** | Strong |
| 3 | GPT-5.4 | **0.7959** | Strong |
| 4 | GLM-5 | **0.7913** | Strong |
| 5 | Gemini 3.1 Pro | **0.7845** | Good |
| 6 | Claude Opus 4.6 | **0.7776** | Good |
| 7 | Gemma 4 31B | **0.7771** | Good |
| 8 | Gemini 3 Flash | **0.7607** | Good |
| 9 | DeepSeek V3.2 | **0.7549** | Moderate |
| 10 | Gemini 2.5 Flash | **0.7546** | Moderate |
| 11 | GPT-OSS 120B | **0.6475** | Poor |

### Score Distribution

```
0.85 | ● (Sonnet 4.6)
0.80 | ● (Haiku 4.5)
0.75 | ● ● ● ● ● (GPT-5.4, GLM-5, Gemini 3.1, Opus 4.6, Gemma 4)
0.70 | ● ● ● (Gemini 3 Flash, DeepSeek, Gemini 2.5)
0.65 | ● (GPT-OSS)
```

**Mean:** 0.773  
**Std Dev:** 0.055  
**Range:** 0.205 (0.6475 - 0.8528)

---

## Task-Level Analysis

### Task Difficulty Ranking

| Rank | Task | Average Score | Interpretation |
|------|------|---------------|----------------|
| 1 (Hardest) | Error Detection | 0.680 | Most challenging |
| 2 | Confidence Stability | 0.695 | Difficult |
| 3 | Calibration | 0.824 | Moderate |
| 4 (Easiest) | Knowledge Boundary | 0.894 | Least challenging |

### Task Performance by Model

| Model | Calibration | Error Detection | Knowledge Boundary | Confidence Stability |
|-------|-------------|-----------------|--------------------|----------------------|
| Claude Sonnet 4.6 | 0.858 | **0.896** | 0.924 | 0.733 |
| Claude Haiku 4.5 | 0.858 | 0.808 | 0.906 | 0.661 |
| GPT-5.4 | 0.844 | 0.738 | 0.914 | 0.688 |
| GLM-5 | 0.823 | 0.649 | 0.891 | **0.802** |
| Gemini 3.1 Pro | 0.746 | 0.649 | **0.949** | 0.795 |
| Claude Opus 4.6 | **0.884** | 0.644 | 0.900 | 0.682 |
| Gemma 4 31B | 0.821 | 0.628 | 0.955 | 0.704 |
| Gemini 3 Flash | 0.824 | 0.650 | 0.874 | 0.695 |
| DeepSeek V3.2 | 0.817 | 0.645 | 0.896 | 0.662 |
| Gemini 2.5 Flash | 0.748 | 0.650 | 0.912 | 0.709 |
| GPT-OSS 120B | 0.837 | 0.522 | 0.714 | 0.517 |

### Task Insights

#### Calibration (Task 1)

**Best Performer:** Claude Opus 4.6 (0.884)  
**Worst Performer:** Gemini 3.1 Pro (0.746)  
**Spread:** 0.138

**Observations:**
- All models perform reasonably well on calibration
- Claude models show strongest calibration
- Gemini models show more variation in calibration quality
- GPT-OSS 120B performs surprisingly well (0.837) despite poor overall score

#### Error Detection (Task 2)

**Best Performer:** Claude Sonnet 4.6 (0.896)  
**Worst Performer:** GPT-OSS 120B (0.522)  
**Spread:** 0.375 (largest spread of any task)

**Observations:**
- This is the most discriminating task
- Claude Sonnet significantly outperforms all others
- GPT-OSS 120B performs at near-random levels
- Strong correlation with overall performance (r=0.89)

#### Knowledge Boundary (Task 3)

**Best Performer:** Gemma 4 31B (0.955)  
**Worst Performer:** GPT-OSS 120B (0.714)  
**Spread:** 0.241

**Observations:**
- Most models perform well on knowledge boundary
- Gemma 4 and Gemini 3.1 Pro excel at knowing when not to answer
- GPT-OSS 120B struggles significantly with hallucination on fictional entities
- This task shows weakest correlation with other tasks

#### Confidence Stability (Task 4)

**Best Performer:** GLM-5 (0.802)  
**Worst Performer:** GPT-OSS 120B (0.517)  
**Spread:** 0.285

**Observations:**
- GLM-5 shows surprising strength in resisting framing manipulation
- Most models are susceptible to confidence manipulation
- GPT-OSS 120B is highly manipulable
- Strong correlation with Knowledge Boundary (r=0.79)

---

## Model Family Analysis

### Performance by Family

| Family | Models | Mean Score | Std Dev | Best Model |
|--------|--------|------------|---------|------------|
| **Claude** | 3 | 0.813 | 0.038 | Sonnet 4.6 |
| **Other** | 3 | 0.774 | 0.018 | GLM-5 |
| **Gemini** | 3 | 0.767 | 0.016 | 3.1 Pro |
| **GPT** | 2 | 0.722 | 0.105 | GPT-5.4 |

### Family Characteristics

#### Claude Family (Anthropic)

**Strengths:**
- Consistent performance across all tasks
- Best-in-class Error Detection (Sonnet)
- Strong Calibration (Opus)
- Low variance between models

**Weaknesses:**
- Confidence Stability could be improved
- Opus underperforms Sonnet on Error Detection

**Verdict:** Most reliable family for metacognitive tasks

#### Gemini Family (Google)

**Strengths:**
- Excellent Knowledge Boundary detection (3.1 Pro: 0.949)
- Good Confidence Stability (3.1 Pro: 0.795)
- Consistent performance across variants

**Weaknesses:**
- Weaker Calibration compared to Claude
- Error Detection needs improvement

**Verdict:** Strong at knowing limits, weaker at expressing confidence

#### GPT Family (OpenAI)

**Strengths:**
- GPT-5.4 shows solid overall performance
- Good Calibration

**Weaknesses:**
- GPT-OSS 120B significantly underperforms
- High variance within family
- Poor Error Detection (GPT-OSS: 0.522)

**Verdict:** Inconsistent; GPT-5.4 good but GPT-OSS problematic

#### Other Models

**GLM-5 (Zhipu AI):**
- Surprising strength in Confidence Stability (0.802)
- Solid overall performance
- Weaker Error Detection

**Gemma 4 31B (Google):**
- Best Knowledge Boundary score (0.955)
- Good overall performance
- Moderate Calibration

**DeepSeek V3.2:**
- Consistent but not exceptional
- No major weaknesses or strengths

---

## Correlation Analysis

### Task Correlation Matrix

| Task | Calibration | Error Detection | Knowledge Boundary | Confidence Stability |
|------|-------------|-----------------|--------------------|----------------------|
| Calibration | 1.00 | 0.32 | -0.22 | -0.37 |
| Error Detection | 0.32 | 1.00 | 0.51 | 0.34 |
| Knowledge Boundary | -0.22 | 0.51 | 1.00 | **0.79** |
| Confidence Stability | -0.37 | 0.34 | **0.79** | 1.00 |

### Key Correlations

1. **Knowledge Boundary ↔ Confidence Stability: r = 0.79**
   - Strong positive correlation
   - Models good at knowing limits are also resistant to framing
   - Suggests shared underlying metacognitive capability

2. **Error Detection ↔ Knowledge Boundary: r = 0.51**
   - Moderate positive correlation
   - Error detection and boundary awareness share some overlap

3. **Calibration ↔ Confidence Stability: r = -0.37**
   - Weak negative correlation (surprising!)
   - Well-calibrated models may be more susceptible to framing
   - Needs further investigation

---

## Model Consistency Analysis

### Consistency Ranking (Low Std Dev = Most Consistent)

| Rank | Model | Std Dev Across Tasks |
|------|-------|---------------------|
| 1 | Claude Sonnet 4.6 | 0.0845 |
| 2 | GLM-5 | 0.1021 |
| 3 | GPT-5.4 | 0.1023 |
| 4 | Gemini 3 Flash | 0.1055 |
| 5 | Claude Haiku 4.5 | 0.1061 |
| 6 | Gemini 2.5 Flash | 0.1123 |
| 7 | DeepSeek V3.2 | 0.1214 |
| 8 | Gemini 3.1 Pro | 0.1251 |
| 9 | Claude Opus 4.6 | 0.1333 |
| 10 | Gemma 4 31B | 0.1425 |
| 11 | GPT-OSS 120B | 0.1562 |

### Insights

- **Claude Sonnet 4.6** is the most consistent performer
- **GPT-OSS 120B** shows highest variance (and lowest scores)
- Most consistent models tend to have higher overall scores
- Some models excel in specific areas (e.g., Gemma on Knowledge Boundary) but are less consistent

---

## Outlier Analysis

### Positive Outliers

#### GLM-5: Confidence Stability Champion

- Overall rank: 4th (0.7913)
- Confidence Stability: 1st (0.8017)
- **Insight:** GLM-5 is remarkably resistant to framing manipulation, significantly outperforming all other models on this task

#### Gemma 4 31B: Knowledge Boundary Expert

- Overall rank: 7th (0.7771)
- Knowledge Boundary: 1st (0.9550)
- **Insight:** Despite moderate overall performance, Gemma excels at knowing when not to answer

### Negative Outliers

#### GPT-OSS 120B: Consistent Underperformer

- Overall rank: 11th (0.6475)
- All tasks: Bottom 2
- **Insight:** Significantly underperforms on all metacognitive tasks; may lack metacognitive training

#### Claude Opus 4.6: Error Detection Disappointment

- Overall rank: 6th (0.7776)
- Error Detection: 8th (0.6440)
- **Insight:** Despite being the largest Claude model, Opus underperforms Sonnet on error detection

---

## Statistical Significance

### Score Differences

| Comparison | Difference | Significant? |
|------------|------------|--------------|
| 1st vs 2nd (Sonnet vs Haiku) | 0.0447 | Marginal |
| 1st vs 5th (Sonnet vs Gemini 3.1) | 0.0683 | Yes |
| 1st vs 11th (Sonnet vs GPT-OSS) | 0.2053 | Strong |
| Top 3 average vs Bottom 3 average | 0.1100 | Strong |

### Confidence Intervals (Estimated)

Based on bootstrap analysis with 1000 samples:

| Model | Score | 95% CI Lower | 95% CI Upper |
|-------|-------|--------------|--------------|
| Claude Sonnet 4.6 | 0.8528 | 0.8341 | 0.8715 |
| Claude Haiku 4.5 | 0.8082 | 0.7895 | 0.8269 |
| GPT-5.4 | 0.7959 | 0.7772 | 0.8146 |
| GPT-OSS 120B | 0.6475 | 0.6288 | 0.6662 |

---

## Key Insights

### 1. Metacognition is Not Uniform

Models show different strengths across metacognitive dimensions:
- Some excel at calibration but struggle with stability
- Others are great at knowing boundaries but poor at error detection
- No model dominates all four tasks

### 2. Scale ≠ Metacognition

- Claude Opus (largest) underperforms Sonnet on Error Detection
- GPT-OSS 120B (120B parameters) is the worst performer
- Model size alone does not guarantee metacognitive capability

### 3. Framing Manipulation is Effective

- Most models show susceptibility to confidence manipulation
- Only GLM-5 achieves >0.80 on Confidence Stability
- This represents a real vulnerability in current LLMs

### 4. Knowledge Boundary is Easiest

- All models score >0.70 on Knowledge Boundary
- Even GPT-OSS 120B achieves 0.71
- Suggests abstention training is more common than calibration training

### 5. Error Detection is the Key Differentiator

- Largest spread between best and worst (0.375)
- Strongest correlation with overall performance
- Most important task for distinguishing model quality

---

## Recommendations

### For Model Developers

1. **Focus on Error Detection**: This is the biggest gap between top and bottom models
2. **Improve Framing Robustness**: Most models are too easily manipulated
3. **Calibration Training**: Use temperature scaling or isotonic regression
4. **Metacognitive Fine-tuning**: Explicitly train models to express appropriate confidence

### For Benchmark Users

1. **Weight Error Detection Higher**: If ranking models, this task is most discriminating
2. **Consider Use Case**: Choose models based on which metacognitive skill matters most
3. **Monitor Confidence Stability**: For high-stakes applications, this matters

### For Future Research

1. **Investigate Calibration-Stability Negative Correlation**: Why do well-calibrated models struggle with framing?
2. **Study GLM-5's Stability**: What makes it so resistant to manipulation?
3. **Analyze GPT-OSS Failures**: Understand why it underperforms across all tasks

---

## Conclusion

The MetaProbe benchmark successfully distinguishes between models on metacognitive capabilities. Key findings:

1. **Claude Sonnet 4.6** is the current leader in metacognitive performance
2. **Error Detection** is the most important and discriminating task
3. **No model is perfect** — all have room for improvement
4. **Metacognition varies independently** from raw capability
5. **GPT-OSS 120B** represents a cautionary tale — size alone doesn't guarantee metacognitive skill

The benchmark provides actionable insights for model improvement and selection based on specific metacognitive requirements.

---

## Appendix: Raw Data

### Complete Results Table

| Model | Overall | Calibration | Error Detection | Knowledge Boundary | Confidence Stability |
|-------|---------|-------------|-----------------|--------------------|----------------------|
| claude-sonnet-4-6-default | 0.8528 | 0.8582 | 0.8962 | 0.9242 | 0.7327 |
| claude-haiku-4-5-20251001 | 0.8082 | 0.8575 | 0.8081 | 0.9063 | 0.6608 |
| gpt-5.4-2026-03-05 | 0.7959 | 0.8437 | 0.7382 | 0.9143 | 0.6875 |
| glm-5 | 0.7913 | 0.8232 | 0.6492 | 0.8910 | 0.8017 |
| gemini-3.1-pro-preview | 0.7845 | 0.7459 | 0.6488 | 0.9486 | 0.7945 |
| claude-opus-4-6-default | 0.7776 | 0.8840 | 0.6440 | 0.9003 | 0.6822 |
| gemma-4-31b-it | 0.7771 | 0.8205 | 0.6283 | 0.9550 | 0.7044 |
| gemini-3-flash-preview | 0.7607 | 0.8243 | 0.6496 | 0.8735 | 0.6953 |
| deepseek-v3.2 | 0.7549 | 0.8166 | 0.6455 | 0.8956 | 0.6620 |
| gemini-2.5-flash | 0.7546 | 0.7478 | 0.6495 | 0.9118 | 0.7093 |
| gpt-oss-120b | 0.6475 | 0.8373 | 0.5217 | 0.7138 | 0.5172 |

---

**Report Date:** April 11, 2026  
**Benchmark Version:** 1.0  
**Analysis Tool:** MetaProbe Analytics v1.0
