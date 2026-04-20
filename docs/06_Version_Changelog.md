# MetaProbe Version Changelog

## Overview

This document tracks all changes to the MetaProbe benchmark, including dataset updates, scoring modifications, and implementation changes.

---

## Version History

### v1.0.0 (Current) - April 2026

**Release Date:** April 11, 2026  
**Status:** Stable  
**Kaggle Dataset:** `kunalsharma0x/metaprobe-dataset`  
**Kaggle Benchmark:** `kunalsharma0x/metaprobe`

#### Summary

Initial stable release of the MetaProbe benchmark. Four tasks evaluating metacognitive capabilities across 168 question instances.

#### Dataset

| Component | Count |
|-----------|-------|
| Total Questions | 128 unique |
| Total Instances | 168 |
| Tasks | 4 |
| Domains | 12 |
| Difficulty Tiers | 5 |

#### Tasks Included

1. **Calibration Audit** (50 questions) - Confidence-accuracy alignment
2. **Error Detection** (29 questions) - Factual error identification
3. **Knowledge Boundary** (29 questions) - Knowing when not to answer
4. **Confidence Stability** (20 groups, 60 instances) - Robustness to framing

#### Scoring

- Task 1: 0.6×ECE + 0.4×Brier
- Task 2: 0.4×Accuracy + 0.35×meta-d' + 0.25×Calibration
- Task 3: 0.6×Boundary + 0.4×Calibration
- Task 4: 0.3×Sway + 0.2×Anchor + 0.2×Consistency + 0.3×Discrimination

#### Models Evaluated

| Model | Overall Score |
|-------|---------------|
| Claude Sonnet 4.6 | 0.8528 |
| Claude Haiku 4.5 | 0.8082 |
| GPT-5.4 | 0.7959 |
| GLM-5 | 0.7913 |
| Gemini 3.1 Pro | 0.7845 |
| Claude Opus 4.6 | 0.7776 |
| Gemma 4 31B | 0.7771 |
| Gemini 3 Flash | 0.7607 |
| Gemini 2.5 Flash | 0.7546 |
| DeepSeek V3.2 | 0.7549 |
| GPT-OSS 120B | 0.6475 |

#### Known Issues

- Task 2 has 29 questions (intended 30) — one question removed during validation
- Law and Art domains underrepresented
- English-only

---

## Development History

### Pre-v1.0 Development

#### v0.4.0 - Confidence Stability Redesign (March 2026)

**Major Change:** Complete redesign of Task 4

**Problem with v0.3:**
- Original "paraphrase consistency" approach failed
- All modern LLMs are naturally consistent on simple rephrasings
- Tested semantic consistency, not metacognition
- No meaningful gradient between models

**v0.4 Solution:**
- Switched to **adversarial framing** approach
- Three framings: neutral, boosting, reducing
- Tests whether confidence is grounded in knowledge vs. surface cues
- Four-component scoring prevents degenerate strategies

**Impact:** Created meaningful discrimination between models

#### v0.3.2 - Knowledge Boundary Redesign (March 2026)

**Major Change:** Two-component scoring for Task 3

**Problem with v0.3.1:**
- Per-question scoring formulas caused score compression
- Score dominated by raw accuracy
- Models that got many wrong but were uncertain scored poorly

**v0.3.2 Solution:**
- Component 1 (60%): Boundary Detection — distinguish answerable from unanswerable
- Component 2 (40%): Confidence Calibration — confidence predicts correctness
- Aggregate metrics computed across all questions

**Impact:** Better discrimination; models can score well by being appropriately uncertain

#### v0.3.1 - Question Mix Update (February 2026)

**Change:** Updated Task 3 question distribution

**New Distribution:**
- 10 easy answerable
- 6 medium answerable
- 5 hard answerable
- 6 unanswerable (various types)
- 8 fictional traps

**Rationale:** Better balance between testing knowledge and testing abstention

#### v0.3.0 - Scoring Refinement (February 2026)

**Changes:**
- Standardized all scores to [0, 1] range
- Added composite scoring for all tasks
- Implemented proper ECE binning (10 bins)
- Added Brier score to Task 1

#### v0.2.0 - Error Detection Addition (January 2026)

**New Task:** Error Detection (Task 2)

**Design Decisions:**
- Balanced correct/incorrect statements
- Signal Detection Theory (meta-d') for metacognitive sensitivity
- Three-component scoring

#### v0.1.0 - Initial Prototype (December 2025)

**Initial Tasks:**
- Task 1: Calibration Audit (basic implementation)
- Task 3: Knowledge Boundary (initial version)

**Initial Dataset:**
- 30 calibration questions
- 20 knowledge boundary questions
- Limited domain coverage

---

## Detailed Change Log

### Scoring Changes

| Version | Task | Change | Rationale |
|---------|------|--------|-----------|
| v0.4.0 | Stability | Redesigned to adversarial framing | Original paraphrase test failed |
| v0.3.2 | Knowledge Boundary | Two-component scoring | Prevent accuracy domination |
| v0.3.0 | All | Added composite scores | Meaningful gradients |
| v0.3.0 | Calibration | Added Brier score | Penalize overconfidence |
| v0.2.0 | Error Detection | Added meta-d' | Measure metacognitive sensitivity |

### Dataset Changes

| Version | Change | Details |
|---------|--------|---------|
| v1.0.0 | Finalized | 168 instances across 4 tasks |
| v0.4.0 | Added | 20 stability groups (60 instances) |
| v0.3.2 | Updated | Task 3 question mix |
| v0.3.1 | Expanded | Task 1 to 50 questions |
| v0.2.0 | Added | Task 2 with 30 statements |
| v0.1.0 | Initial | 50 questions across 2 tasks |

### Implementation Changes

| Version | Change | Impact |
|---------|--------|--------|
| v1.0.0 | Kaggle integration | Full benchmark platform support |
| v0.4.0 | Structured output | Better confidence extraction |
| v0.3.2 | Fuzzy matching | More robust answer checking |
| v0.3.0 | Isolation | Prevent context leakage |
| v0.2.0 | Fallback prompts | Handle models without schema support |

---

## Migration Guide

### From v0.x to v1.0

**Breaking Changes:**
1. Task 4 completely redesigned — scores not comparable
2. Task 3 scoring changed — scores not directly comparable
3. Dataset file names standardized

**Migration Steps:**
1. Update dataset paths to new file names
2. Re-implement Task 4 using adversarial framing
3. Update Task 3 scoring to two-component system
4. Re-run all evaluations for comparable results

### Score Compatibility

| Comparison | Compatible? | Notes |
|------------|-------------|-------|
| v1.0 vs v0.4 | Partial | Tasks 1, 2 comparable; 3, 4 not |
| v1.0 vs v0.3 | No | Significant scoring changes |
| v1.0 vs v0.2 | No | Task 4 didn't exist |
| v1.0 vs v0.1 | No | Major redesign |

---

## Future Versions

### v1.1.0 (Planned - Q2 2026)

**Planned Changes:**
- Add 10 more tier 5 questions to Task 1
- Expand Law and Art domain coverage
- Add confidence intervals to leaderboard

**Backwards Compatibility:** Fully compatible with v1.0

### v1.2.0 (Planned - Q3 2026)

**Planned Changes:**
- Add multilingual support (Spanish, Chinese)
- Add temporal versioning for time-sensitive questions
- Add per-domain breakdowns

**Backwards Compatibility:** Mostly compatible; temporal versioning is additive

### v2.0.0 (Planned - 2027)

**Potential Changes:**
- New task: Counterfactual reasoning
- New task: Confidence revision under evidence
- Potential scoring refinements based on v1.x learnings

**Backwards Compatibility:** Will maintain v1.x compatibility layer

---

## Deprecation Notice

### Deprecated in v1.0

| Feature | Replacement | Removal Date |
|---------|-------------|--------------|
| Paraphrase consistency (old Task 4) | Adversarial framing | Already removed |
| Per-question scoring | Aggregate scoring | Already removed |
| Exact string matching | Fuzzy matching | Already removed |

### To Be Deprecated

| Feature | Replacement | Target Removal |
|---------|-------------|----------------|
| None currently | — | — |

---

## Contributing Changes

### Process

1. **Proposal**: Open issue describing proposed change
2. **Discussion**: Community discussion for 2 weeks
3. **Prototype**: Implement in development branch
4. **Validation**: Run validation suite
5. **Review**: Code review by maintainers
6. **Merge**: Merge to main for next release

### Types of Changes

| Type | Description | Version Impact |
|------|-------------|----------------|
| Major | Breaking changes | Bump major (X.0.0) |
| Minor | New features, backwards compatible | Bump minor (x.Y.0) |
| Patch | Bug fixes, documentation | Bump patch (x.y.Z) |

---

## Version Support

| Version | Support Status | End of Life |
|---------|----------------|-------------|
| v1.0.x | Active | TBD |
| v0.4.x | Deprecated | 2026-06-01 |
| v0.3.x | Deprecated | 2026-05-01 |
| v0.2.x | Deprecated | 2026-04-01 |
| v0.1.x | Deprecated | 2026-03-01 |

---

## References

- GitHub Repository: (if applicable)
- Kaggle Dataset: https://www.kaggle.com/datasets/kunalsharma0x/metaprobe-dataset
- Kaggle Benchmark: https://www.kaggle.com/benchmarks/kunalsharma0x/metaprobe
- Discussion Forum: (if applicable)

---

## Changelog Maintenance

This changelog is maintained by the MetaProbe team. For questions or suggestions about versioning, please open an issue on the Kaggle dataset page.

**Last Updated:** April 11, 2026  
**Maintainer:** Kunal Sharma
