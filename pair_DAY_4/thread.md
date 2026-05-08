# Tweet Thread — Day 4: Evaluation and Statistics

---

**Tweet 1:**
You ran a paired bootstrap test.
Got p=0.189.
Wrote "DO NOT DEPLOY" in your memo.

But do you know what p=0.189 actually means mathematically?
Or why paired bootstrap — not t-test, not Wilcoxon?

Here's the full mechanism. 🧵

---

**Tweet 2:**
Why NOT a paired t-test?

t-test assumes your score differences are normally
distributed. Agent benchmark scores are bounded 0-1,
skewed, and contain discrete jumps from rule-based
components.

Paired bootstrap makes zero distributional assumptions.
It builds its null distribution directly from YOUR data.

That's why it's the correct test for NLP evaluation.
(Dror et al. ACL 2018)

---

**Tweet 3:**
How paired bootstrap constructs its null distribution:

Step 1: Compute observed delta (mean score difference
across 59 tasks)
Step 2: Resample 59 differences WITH replacement
Step 3: Repeat 10,000 times → bootstrap distribution
Step 4: Shift distribution to center at 0 (null hypothesis)
Step 5: p-value = proportion of shifted deltas ≥ observed

Real output (numpy, seed=42, n=59):
Observed Delta: 0.0239
Bootstrap std: 0.0175
p-value: 0.0856
95% CI: [-0.0107, 0.0577]

---

**Tweet 4:**
What p=0.189 actually means:

18.9% of bootstrap resamples showed a delta as large
as yours under the null hypothesis.

NOT: "the trained model is worse"
NOT: "Delta A = 0"
NOT: "your training failed"

YES: "with n=59 tasks, your effect size is too small
relative to score variance to rule out chance"

Your DO NOT DEPLOY signal is correct.
The reason is underpowered detection — not proven failure.

---

**Tweet 5:**
The hidden problem: your 60/40 blending formula.

condition_trained_judge uses:
final_score = 0.6 _ judge_score + 0.4 _ machine_score

Machine scores are deterministic — low variance.
Blending compresses the score distribution.

Measured variance reduction: 36.0%

But baseline uses pure machine scores.
You're comparing two different variance regimes.
Your bootstrap is not comparing apples to apples.

Fix: blend ALL conditions consistently — or compare
pure judge scores only.

---

**Tweet 6:**
Full explainer with step-by-step bootstrap code,
real verified output, power analysis for n=59,
and the 60/40 blending variance reduction diagnostic:

https://yakobdereje.substack.com/p/why-paired-bootstrap-is-the-right?r=8bqorb

Sources:

- Dror et al. 2018 (ACL) — aclanthology.org/P18-1128/
- Efron & Tibshirani 1993 — An Introduction to the Bootstrap
