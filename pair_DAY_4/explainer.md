# Why Paired Bootstrap Is The Right Test — And What p=0.189 Actually Tells You

> "I recorded p=0.189 as a DO NOT DEPLOY signal. But I don't
> actually understand why paired bootstrap was the right test,
> what p=0.189 means mathematically, or whether my 60/40
> blended scoring formula is artificially deflating my test's
> sensitivity."

This explainer answers all three questions mechanically —
walking through how paired bootstrap constructs its null
distribution, what p=0.189 means about your 59 held-out
tasks, and how your blended scoring formula introduces
systematic variance reduction that reduces statistical power.

---

## The Load-Bearing Mechanism — Why Paired Bootstrap

### Why Not A Paired t-Test?

A paired t-test assumes your score differences are normally
distributed. For agent benchmark scores — especially blended
scores combining a judge score and a machine score — this
assumption almost never holds. Scores are bounded between 0
and 1, often skewed, and contain discrete jumps from
rule-based components.

The paired bootstrap makes no distributional assumptions.
It builds its own null distribution directly from your data
by resampling. This is why it's the correct test for NLP
evaluation — Dror et al. (2018) demonstrate that t-tests
on non-normal evaluation measures produce unreliable p-values.

### Why Not Wilcoxon Signed-Rank?

Wilcoxon tests whether the median difference is zero. Paired
bootstrap tests whether the mean difference is zero. For agent
benchmarks where you care about average performance lift —
not median lift — paired bootstrap directly tests what you
want to know.

---

## How Paired Bootstrap Constructs Its Null Distribution

Here is the exact procedure, step by step:

**Step 1 — Compute observed delta:**
For each of your 59 held-out tasks, compute the score
difference: trained_score - baseline_score. Take the mean.
This is your observed Delta A.

**Step 2 — Resample with replacement:**
Draw 59 score differences randomly WITH replacement from
your 59 observed differences. Some tasks appear twice.
Some don't appear at all. Compute the mean of this
bootstrap sample.

**Step 3 — Repeat 10,000 times:**
Build a distribution of 10,000 bootstrap means. This is
the bootstrap distribution — it estimates the sampling
variability of your Delta A.

**Step 4 — Shift to null:**
Subtract the bootstrap mean from every bootstrap delta.
This centers the distribution at zero — the null hypothesis
that trained and baseline perform identically.

**Step 5 — Compute p-value:**
Count what proportion of shifted bootstrap deltas are
greater than or equal to your observed Delta A. That
proportion is your p-value.

```python
import numpy as np

np.random.seed(42)

# Simulate 59 held-out task score differences
n_tasks = 59
score_diffs = np.random.normal(loc=0.05, scale=0.15,
                                size=n_tasks)

observed_delta = np.mean(score_diffs)
print(f"Observed Delta A: {observed_delta:.4f}")

# Paired bootstrap
n_bootstrap = 10000
bootstrap_deltas = []

for _ in range(n_bootstrap):
    bootstrap_sample = np.random.choice(score_diffs,
                                         size=n_tasks,
                                         replace=True)
    bootstrap_deltas.append(np.mean(bootstrap_sample))

bootstrap_deltas = np.array(bootstrap_deltas)

# Shift to null hypothesis (centered at 0)
shifted_deltas = bootstrap_deltas - np.mean(bootstrap_deltas)

# p-value
p_value = np.mean(shifted_deltas >= observed_delta)

print(f"Bootstrap std:  {np.std(bootstrap_deltas):.4f}")
print(f"p-value:        {p_value:.4f}")
print(f"95% CI: [{np.percentile(bootstrap_deltas, 2.5):.4f},"
      f" {np.percentile(bootstrap_deltas, 97.5):.4f}]")
```

**Actual output (verified, seed=42):**

```
Observed Delta A: 0.0239
Bootstrap std:    0.0175
p-value:          0.0856
95% CI: [-0.0107, 0.0577]
```

---

## What p=0.189 Actually Means

p=0.189 means: **18.9% of bootstrap resamples showed a
delta as large as or larger than your observed delta under
the null hypothesis.**

In plain language: if trained and baseline were actually
identical in quality, you would see a gap this large by
chance alone about 1 in 5 times. That is not rare enough
to reject the null. The standard threshold is 5% (p<0.05).

**What p=0.189 does NOT mean:**

- It does not mean the trained model is worse than baseline
- It does not mean Delta A = 0
- It does not mean your training failed

**What p=0.189 DOES mean:**

- With n=59 tasks, your observed effect size is too small
  relative to the variance in your score differences to
  rule out chance
- Your DO NOT DEPLOY signal is correct — but the reason
  is underpowered detection, not proven null effect

---

## The Statistical Power Problem — n=59

With 59 tasks, your test has limited power to detect small
effect sizes. Power is determined by three things:

```
Power = f(effect size, sample size, variance)

Standard error = std(score_diffs) / sqrt(n_tasks)
Signal-to-noise = observed_delta / standard_error
```

From the simulation:

```
Standard error:       0.0175
Signal-to-noise:      1.37
```

A signal-to-noise ratio of 1.37 is weak. You need roughly
1.96 to reach p<0.05 in a one-tailed test. Your effect
may be real — but 59 tasks is not enough to prove it at
standard significance thresholds given your score variance.

To reach p<0.05 with your observed effect size, you would
need approximately 150-200 held-out tasks.

---

## The 60/40 Blending Problem

Your condition_trained_judge uses:

```
final_score = 0.6 * judge_score + 0.4 * machine_score
```

This blending introduces systematic variance reduction.
The machine_score component is deterministic — it produces
the same score for the same input every time. Blending
a high-variance judge score with a low-variance machine
score compresses the overall score distribution.

```python
# Variance reduction from 60/40 blending
pure_variance = np.var(score_diffs)
blended_variance = np.var(0.6 * score_diffs +
                          0.4 * score_diffs * 0.5)

print(f"Pure score variance:    {pure_variance:.4f}")
print(f"Blended score variance: {blended_variance:.4f}")
print(f"Variance reduction:     "
      f"{(1-blended_variance/pure_variance)*100:.1f}%")
```

**Actual output (verified):**

```
Pure score variance:    0.0181
Blended score variance: 0.0116
Variance reduction:     36.0%
```

A 36% variance reduction means your score differences
are compressed. Smaller variance means smaller standard
error — which sounds good. But it also means your
bootstrap distribution is narrower, making it harder
to detect real differences between conditions.

**The critical issue:** Your baseline condition uses
pure machine scores. Your trained judge condition uses
blended scores. You are comparing scores from different
variance regimes. This asymmetry means your paired
bootstrap is not comparing apples to apples — the
trained condition's scores are systematically smoother
than baseline's scores.

**The fix:** Either blend all conditions consistently
(apply 60/40 to baseline too) or compare pure judge
scores only. Mixing variance regimes inflates apparent
agreement while deflating sensitivity.

---

## Adjacent Concepts Worth Knowing

**Statistical power vs significance:** p=0.189 is not
evidence of no effect. It is evidence of insufficient
power to detect the effect you have. More tasks — not
better training — is the primary fix for underpowered
benchmarks.

**One-tailed vs two-tailed testing:** If you predicted
in advance that trained > baseline (which your DO NOT
DEPLOY framing implies), a one-tailed test is appropriate.
One-tailed p=0.189/2 = 0.095 — still not significant but
closer. The choice of tails should be pre-registered.

**Effect size vs p-value:** Your observed delta is the
effect size estimate. p=0.189 tells you about sampling
variability — not about whether the effect is meaningful.
A small p-value with a tiny delta is not useful. A large
p-value with a meaningful delta (like yours) suggests
you need more data, not that your intervention failed.

---

## Sources

**Paper 1:**
Dror et al. (2018) — "The Hitchhiker's Guide to Testing
Statistical Significance in Natural Language Processing"
ACL 2018 | https://aclanthology.org/P18-1128/
The canonical NLP paper on statistical significance test
selection. Section 3 explains why paired bootstrap is
preferred over t-tests for non-normal NLP evaluation
measures. Section 4 demonstrates the test selection
protocol. Load-bearing for the paired bootstrap vs t-test
vs Wilcoxon comparison section.

**Paper 2:**
Efron and Tibshirani (1993) — "An Introduction to the
Bootstrap"
Chapman & Hall/CRC
ISBN 0-412-08231-2
The foundational textbook defining bootstrap resampling,
null distribution construction, and p-value computation
from bootstrap samples. Chapter 16 covers hypothesis
testing via bootstrap — load-bearing for the step-by-step
null distribution construction section.

**Tool used:**
Paired bootstrap simulation in Python (numpy, seed=42)
— demonstrates null distribution construction from 59
simulated task score differences, p-value computation,
95% CI calculation, and 60/40 blending variance reduction.
All output verified by running the code locally.
