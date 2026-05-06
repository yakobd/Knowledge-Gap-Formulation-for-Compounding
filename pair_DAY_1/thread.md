# Tweet Thread — Day 1: Inference-Time Mechanics

---

**Tweet 1:**
Your critic hits 90%+ preference accuracy.
You deploy it with rejection sampling.
Lift is near zero.

The problem isn't your critic.
It's structural — and it lives at inference time.

🧵 Thread:

---

**Tweet 2:**
During training, your critic learned on pairs
that were _deliberately_ different.
Chosen vs rejected. Large quality margin.

During deployment, you generate k candidates
from the same model at the same temperature.
They cluster. The margin collapses.

Your critic can't lift what it can't discriminate.

---

**Tweet 3:**
The sommelier analogy:

World-class wine expert = your critic (90%+ accurate).
5 glasses from the same bottle = your k candidates.

Skill is real. Nothing to discriminate.
Lift = zero.

Not a critic failure. A candidate pool failure.

---

**Tweet 4:**
The cheap diagnostic — run this BEFORE retraining:

Measure score_std of your deployment candidate pool.

score_std < 0.1 → GENERATION PROBLEM
Fix: diverse prompts, higher temperature, multiple models.
Do NOT retrain the critic yet.

score_std ≥ 0.1 → CRITIC PROBLEM
Fix: harder negatives, more training data.

---

**Tweet 5:**
The numbers from real simulation (numpy, seed=42):

Training pair avg margin: 1.808
Deployment pool avg margin: 0.145
Ratio: 12.5x

Your critic was trained on pairs 12.5x more
discriminable than what deployment provides.

This is why teams waste weeks collecting training
data when the bottleneck is upstream of the critic.

---

**Tweet 6:**
Full explainer with verified simulation code,
decision rule, and adjacent concepts:

https://yakobdereje.substack.com/p/why-your-critics-90-accuracy-doesnt

Sources:

- Stiennon et al. 2020 (NeurIPS) — arxiv.org/abs/2009.01325
- Lambert et al. 2024 (RewardBench) — arxiv.org/abs/2403.13787
