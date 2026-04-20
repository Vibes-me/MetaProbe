# MetaProbe Model Behavior Taxonomy

## Overview

This document categorizes model behaviors observed on the MetaProbe benchmark. Understanding these behavioral patterns helps identify failure modes and guides improvement strategies.

---

## Taxonomy Structure

Models are categorized along four dimensions:
1. **Calibration Profile** — How well confidence matches accuracy
2. **Error Detection Profile** — Ability to spot factual errors
3. **Boundary Awareness Profile** — Knowing when not to answer
4. **Stability Profile** — Resistance to framing manipulation

---

## Calibration Profiles

### Type A: Well-Calibrated

**Characteristics:**
- Confidence ratings closely track actual accuracy
- ECE < 0.15
- Uses full confidence range (0.1 to 0.9+)

**Examples:**
- Claude Opus 4.6 (ECE score: 0.884)
- Claude Sonnet 4.6 (ECE score: 0.858)

**Behavioral Markers:**
- Says "I'm about 70% confident" and is correct ~70% of the time
- Adjusts confidence based on question difficulty
- Rarely expresses extreme confidence (0.99) unless certain

**Training Implications:**
- Likely trained with calibration objectives
- May use temperature scaling or similar techniques

---

### Type B: Overconfident

**Characteristics:**
- Expresses high confidence even when frequently wrong
- ECE > 0.25
- Confidence clustered at 0.8-1.0 range

**Examples:**
- None prominently in current leaderboard (good sign!)
- Historically common in older LLMs

**Behavioral Markers:**
- Says "I'm 95% confident" but is only correct 60% of the time
- Little variation in confidence across questions
- Tends to express certainty even on difficult questions

**Root Causes:**
- Training on next-token prediction without calibration
- Lack of explicit uncertainty modeling
- Reinforcement learning from human feedback may encourage confidence

**Mitigation Strategies:**
- Temperature scaling on output probabilities
- Calibration-specific fine-tuning
- Explicit confidence prediction training

---

### Type C: Underconfident

**Characteristics:**
- Expresses low confidence even when frequently correct
- ECE > 0.20 but inverted pattern
- Confidence clustered at 0.4-0.7 range

**Examples:**
- Some Gemini models show mild underconfidence

**Behavioral Markers:**
- Says "I'm 60% confident" but is correct 85% of the time
- Conservative confidence expressions
- May reflect training to avoid overcommitment

**Root Causes:**
- Safety training that discourages strong claims
- Uncertainty about evaluation criteria
- Conservative fine-tuning

---

## Error Detection Profiles

### Type I: Accurate Detector

**Characteristics:**
- High accuracy in identifying correct/incorrect statements
- Strong metacognitive sensitivity (meta-d' > 0.7)
- Confidence in judgments is well-calibrated

**Examples:**
- Claude Sonnet 4.6 (Error Detection: 0.896)
- Claude Haiku 4.5 (Error Detection: 0.808)

**Behavioral Markers:**
- Correctly identifies "Water boils at 90°C" as false
- Expresses appropriate confidence in detection judgment
- Can explain why a statement is incorrect

**Capabilities:**
- Strong factual knowledge
- Good metacognitive monitoring
- Ability to cross-check statements against knowledge

---

### Type II: Gullible (Tends to Accept)

**Characteristics:**
- Tends to accept statements as correct
- High false negative rate on incorrect statements
- May lack critical evaluation skills

**Examples:**
- GPT-OSS 120B (Error Detection: 0.522)

**Behavioral Markers:**
- Accepts "Isaac Newton discovered penicillin" as correct
- Low confidence variation between true and false statements
- May be optimized for helpfulness over accuracy

**Root Causes:**
- Training to be agreeable/helpful
- Lack of explicit error detection training
- Optimized for fluency over fact-checking

**Failure Modes:**
- Accepts common misconceptions
- Fails to catch subtle factual errors
- Overly trusting of input statements

---

### Type III: Skeptical (Tends to Reject)

**Characteristics:**
- Tends to reject statements as incorrect
- High false positive rate on correct statements
- May be overly cautious

**Examples:**
- None prominent in current results

**Behavioral Markers:**
- Rejects true statements due to minor phrasing differences
- Expresses doubt even on clearly correct facts
- May reflect conservative training

---

## Boundary Awareness Profiles

### Type X: Clear Boundary

**Characteristics:**
- Appropriately attempts answerable questions
- Abstains on unanswerable/fictional questions
- Low hallucination rate on fictional entities

**Examples:**
- Gemma 4 31B (Knowledge Boundary: 0.955)
- Gemini 3.1 Pro (Knowledge Boundary: 0.949)

**Behavioral Markers:**
- Answers "Capital of France" confidently
- Abstains on "Republic of Carpathia" (fictional country)
- Says "I don't know" for future events

**Capabilities:**
- Strong self-knowledge
- Appropriate abstention thresholds
- Resistance to hallucination

---

### Type Y: Overconfident Boundary

**Characteristics:**
- Attempts to answer unanswerable questions
- Hallucinates on fictional entities
- Poor abstention judgment

**Examples:**
- GPT-OSS 120B (Knowledge Boundary: 0.714)

**Behavioral Markers:**
- Makes up information about fictional countries
- Attempts to answer questions about future events
- Rarely says "I don't know"

**Root Causes:**
- Training to always provide an answer
- Lack of abstention training
- Optimized for engagement over accuracy

**Failure Modes:**
- Hallucinates details about fictional entities
- Provides confident but wrong answers to unanswerable questions
- Fails to recognize knowledge limits

---

### Type Z: Overly Cautious Boundary

**Characteristics:**
- Abstains even on answerable questions
- Underconfident in knowledge
- May be too conservative

**Examples:**
- None prominent in current results

**Behavioral Markers:**
- Says "I don't know" for common knowledge questions
- Low confidence on easy questions
- May frustrate users with excessive caution

---

## Stability Profiles

### Type S: Stable (Resistant to Framing)

**Characteristics:**
- Confidence remains consistent across framings
- Not swayed by boosting or reducing language
- Confidence reflects actual knowledge

**Examples:**
- GLM-5 (Confidence Stability: 0.802)
- Gemini 3.1 Pro (Confidence Stability: 0.795)

**Behavioral Markers:**
- Same confidence for "What is X?" and "You surely know what X is"
- Not intimidated by "experts debate this" framing
- Confidence changes < 0.1 between framings

**Capabilities:**
- Genuine knowledge-based confidence
- Immunity to social manipulation
- Stable self-assessment

---

### Type M: Manipulable (Swayed by Framing)

**Characteristics:**
- Confidence swings with framing
- Boosted by encouraging language
- Reduced by doubt-inducing language

**Examples:**
- GPT-OSS 120B (Confidence Stability: 0.517)
- Claude Haiku 4.5 (Confidence Stability: 0.661)

**Behavioral Markers:**
- Confidence increases 0.2+ with boosting framing
- Confidence decreases 0.2+ with reducing framing
- May change answer based on framing

**Root Causes:**
- Confidence derived from prompt patterns, not knowledge
- Social conditioning in training
- Lack of robust confidence mechanisms

**Vulnerabilities:**
- Susceptible to social engineering
- Can be manipulated into overconfidence
- Unreliable in adversarial settings

---

### Type D: Discriminating but Unstable

**Characteristics:**
- Good discrimination (higher confidence when correct)
- But confidence varies with framing
- Mixed profile

**Examples:**
- Several mid-tier models show this pattern

**Behavioral Markers:**
- Correctly more confident on easy vs hard questions
- But still swayed by framing within difficulty level
- Partial metacognitive capability

---

## Combined Behavioral Archetypes

### Archetype 1: The Expert (Claude Sonnet 4.6)

**Profile:**
- Calibration: Well-Calibrated (A)
- Error Detection: Accurate Detector (I)
- Boundary: Clear Boundary (X)
- Stability: Moderately Stable (S-)

**Description:**
Strong across all dimensions with slight susceptibility to framing. The current gold standard for metacognitive capabilities.

**Strengths:**
- Excellent error detection
- Good calibration
- Appropriate boundaries

**Weaknesses:**
- Mild framing susceptibility

---

### Archetype 2: The Cautious Scholar (Claude Haiku 4.5)

**Profile:**
- Calibration: Well-Calibrated (A)
- Error Detection: Accurate Detector (I)
- Boundary: Clear Boundary (X)
- Stability: Manipulable (M)

**Description:**
Reliable and accurate but more susceptible to framing than Sonnet. Good for applications where stability is less critical.

**Strengths:**
- Good calibration
- Strong error detection
- Appropriate boundaries

**Weaknesses:**
- Significant framing susceptibility

---

### Archetype 3: The Confident Generalist (GPT-5.4)

**Profile:**
- Calibration: Well-Calibrated (A)
- Error Detection: Moderate Detector
- Boundary: Clear Boundary (X)
- Stability: Moderately Stable (S-)

**Description:**
Solid all-around performer with good calibration and boundaries. Error detection is good but not exceptional.

**Strengths:**
- Good calibration
- Appropriate boundaries
- Decent stability

**Weaknesses:**
- Error detection could be improved

---

### Archetype 4: The Stable Specialist (GLM-5)

**Profile:**
- Calibration: Well-Calibrated (A)
- Error Detection: Moderate Detector
- Boundary: Clear Boundary (X)
- Stability: Highly Stable (S+)

**Description:**
Exceptional stability with good overall performance. Best choice for applications requiring resistance to manipulation.

**Strengths:**
- Best-in-class stability
- Good calibration
- Appropriate boundaries

**Weaknesses:**
- Error detection is average

---

### Archetype 5: The Boundary Expert (Gemma 4 31B, Gemini 3.1 Pro)

**Profile:**
- Calibration: Moderate
- Error Detection: Moderate Detector
- Boundary: Exceptional Boundary (X+)
- Stability: Highly Stable (S+)

**Description:**
Excels at knowing when not to answer and resists framing. Good for applications where hallucination prevention is critical.

**Strengths:**
- Exceptional boundary awareness
- Good stability
- Low hallucination rate

**Weaknesses:**
- Calibration could be improved
- Error detection is moderate

---

### Archetype 6: The Struggler (GPT-OSS 120B)

**Profile:**
- Calibration: Well-Calibrated (A) — surprisingly!
- Error Detection: Gullible (II)
- Boundary: Overconfident Boundary (Y)
- Stability: Highly Manipulable (M-)

**Description:**
Poor performance across most dimensions despite good calibration. Tends to accept errors, hallucinate on fiction, and is highly manipulable.

**Strengths:**
- Calibration is actually decent

**Weaknesses:**
- Poor error detection
- Hallucinates on fictional entities
- Highly susceptible to framing
- Overall worst performer

**Hypothesis:**
May be a base model without sufficient safety/alignment fine-tuning. Calibration is good because it's trained on probabilities, but lacks metacognitive judgment.

---

## Behavioral Patterns by Task Interaction

### Pattern 1: Calibration-Stability Tradeoff

**Observation:** Models with excellent calibration tend to be more susceptible to framing (negative correlation: r = -0.37)

**Hypothesis:**
- Well-calibrated models may rely more on "confidence heuristics" that are easily manipulated
- Stability requires deeper knowledge-based confidence that may be harder to calibrate

**Implication:**
There's a tension between being well-calibrated and being stable. Future models need to achieve both.

---

### Pattern 2: Boundary-Stability Alignment

**Observation:** Strong positive correlation between Knowledge Boundary and Confidence Stability (r = 0.79)

**Hypothesis:**
- Both tasks require robust self-knowledge
- Models that know their limits are also resistant to manipulation
- Shared underlying metacognitive capability

**Implication:**
Improving boundary awareness may also improve stability, and vice versa.

---

### Pattern 3: Error Detection as General Indicator

**Observation:** Error Detection correlates strongly with overall performance (r = 0.89)

**Hypothesis:**
- Error detection requires multiple capabilities: knowledge, monitoring, judgment
- It's the most "complete" metacognitive task
- Models that excel here tend to excel overall

**Implication:**
Error Detection should be weighted heavily in overall scoring.

---

## Failure Mode Catalog

### FM-1: The Overconfident Hallucinator

**Symptoms:**
- High confidence on fictional entities
- Attempts to answer unanswerable questions
- Calibration appears good (confidence matches accuracy on answerable questions)

**Example:** None in current results (good!)

**Detection:**
- Check Knowledge Boundary score
- Look for high calibration but low boundary

**Mitigation:**
- Explicit abstention training
- Fictional entity exposure during training

---

### FM-2: The Gullible Believer

**Symptoms:**
- Accepts false statements as true
- Low Error Detection score
- May have good calibration on questions it answers

**Example:** GPT-OSS 120B

**Detection:**
- Low Error Detection score
- Tends to accept statements without critical evaluation

**Mitigation:**
- Critical thinking training
- Fact-checking fine-tuning
- Explicit error detection objectives

---

### FM-3: The Manipulable Puppet

**Symptoms:**
- Confidence swings with framing
- Low Confidence Stability score
- May give different answers based on framing

**Example:** GPT-OSS 120B, Claude Haiku 4.5 (to lesser extent)

**Detection:**
- Large confidence variance between boosting/reducing framings
- Answer inconsistency across framings

**Mitigation:**
- Adversarial training with various framings
- Confidence robustness objectives
- Exposure to manipulative prompts during training

---

### FM-4: The Cautious Abstainer

**Symptoms:**
- Abstains on answerable questions
- High "miss" rate on Knowledge Boundary
- May frustrate users with excessive "I don't know"

**Example:** None prominent in current results

**Detection:**
- Low hit rate on answerable questions
- High abstention rate overall

**Mitigation:**
- Encourage more attempts
- Lower abstention thresholds
- Calibration of abstention confidence

---

## Implications for Model Selection

### Use Case: High-Stakes Factual Queries

**Priority:** Error Detection > Calibration > Boundary > Stability

**Recommended Models:**
1. Claude Sonnet 4.6
2. Claude Haiku 4.5
3. GPT-5.4

---

### Use Case: Open-Ended Generation (Low Hallucination)

**Priority:** Boundary > Stability > Calibration > Error Detection

**Recommended Models:**
1. Gemma 4 31B
2. Gemini 3.1 Pro
3. GLM-5

---

### Use Case: Adversarial/Security Context

**Priority:** Stability > Error Detection > Boundary > Calibration

**Recommended Models:**
1. GLM-5
2. Gemini 3.1 Pro
3. Claude Sonnet 4.6

---

### Use Case: General Purpose (Balanced)

**Priority:** Overall Score

**Recommended Models:**
1. Claude Sonnet 4.6
2. Claude Haiku 4.5
3. GPT-5.4

---

## Future Research Directions

1. **Causal Analysis:** What training factors lead to different behavioral profiles?
2. **Intervention Studies:** Can we convert a "Gullible" model to an "Accurate Detector"?
3. **Scaling Analysis:** How do behaviors change with model size within a family?
4. **Temporal Analysis:** Do behaviors change with continued training?
5. **Cross-Lingual:** Do behavioral profiles transfer across languages?

---

## Conclusion

The MetaProbe taxonomy reveals distinct behavioral archetypes among current LLMs. Understanding these profiles enables:

1. **Better Model Selection:** Choose models based on behavioral fit for the use case
2. **Targeted Improvement:** Focus training on specific weaknesses
3. **Failure Prediction:** Anticipate problems based on behavioral profile
4. **Benchmark Design:** Create more targeted evaluations for specific behaviors

The field currently lacks a model that excels across all behavioral dimensions — there remains significant room for improvement in metacognitive capabilities.

---

**Document Version:** 1.0  
**Last Updated:** April 2026  
**Based On:** MetaProbe Benchmark v1.0 Results
