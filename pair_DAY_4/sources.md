# Sources — Day 4

**Explainer written by:** Yakob Dereje
**For question asked by:** Rafia Kedir

---

## Canonical Paper 1

**Dror et al. (2018) — "The Hitchhiker's Guide to Testing
Statistical Significance in Natural Language Processing"**
ACL 2018
https://aclanthology.org/P18-1128/

The canonical NLP paper on statistical significance test
selection. Section 3 explains why paired bootstrap is
preferred over t-tests for non-normal NLP evaluation
measures. Section 4 demonstrates the test selection
protocol. Load-bearing for the paired bootstrap vs t-test
vs Wilcoxon comparison section and the null distribution
construction explanation.

---

## Canonical Paper 2

**Efron and Tibshirani (1993) — "An Introduction to
the Bootstrap"**
Chapman and Hall/CRC
ISBN 0-412-08231-2

The foundational textbook defining bootstrap resampling,
null distribution construction, and p-value computation
from bootstrap samples. Chapter 16 covers hypothesis
testing via bootstrap — load-bearing for the step-by-step
null distribution construction and the p-value
interpretation section.

---

## Tool / Pattern Used

**Paired bootstrap simulation — Python (numpy, seed=42)**

Simulated 59 held-out task score differences, constructed
bootstrap null distribution across 10,000 resamples,
computed p-value, 95% CI, signal-to-noise ratio, and
60/40 blending variance reduction (36%). All output
verified by running the code locally. Full simulation
in explainer.md.

---

## Follow-On Reading

**Dror et al. (2020) — "Statistical Significance Testing
for Natural Language Processing"**
Morgan and Claypool
The book-length expansion of the 2018 ACL paper — covers
power analysis, multiple comparisons, and cross-validation
evaluation. Essential next read for anyone building NLP
benchmarks who needs to report defensible statistical
results.
