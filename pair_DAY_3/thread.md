# Tweet Thread — Day 3: Training and Post-Training Mechanics

---

**Tweet 1:**
Your preference training improved benchmark scores.
But did your model learn to reason about evidence —
or just learn to say "scope", "validate", "phased approach"?

These look identical on a benchmark.
They are completely different in production.

Here's how to tell. 🧵

---

**Tweet 2:**
Every preference objective — DPO, SimPO, ORPO — does
one thing: shifts token probability distributions.

The question is WHAT shifts and WHY.

DPO: shifts relative to a reference model (anchored)
SimPO: shifts without reference, normalized by length
ORPO: shifts while simultaneously learning from positives

Same goal. Different mechanisms. Different gaming risks.

---

**Tweet 3:**
Real simulation showing the difference (numpy, seed=42):

Before training → After genuine learning → After gaming

scope: 0.100 → 0.115 → 0.202
validate: 0.088 → 0.103 → 0.191
evidence: 0.112 → 0.231 → 0.101
discovery: 0.100 → 0.205 → 0.090

Genuine learning: evidence and discovery rise most.
Evaluator gaming: scope and validate rise most.

The model learned different things. Same benchmark score.

---

**Tweet 4:**
The diagnostic — run this BEFORE reporting results:

Generate responses to two prompts:

- Strong evidence: budget confirmed, timeline realistic
- Weak evidence: no budget, aggressive timeline

Count cautious tokens in each response.
Compute ratio: weak_cautious / strong_cautious

ratio > 2.0 → Genuine reasoning ✅
ratio ≈ 1.0 → Evaluator gaming ⚠️

If the model hedges on both — it learned surface tokens,
not evidence-conditional reasoning.

---

**Tweet 5:**
Why does this happen mechanically?

Preference training optimizes whatever pattern separates
chosen from rejected responses.

If chosen responses consistently contain "scope" —
the model raises P("scope") across ALL contexts.
Not when evidence is weak. Always.

The quality of your preference pairs determines
whether you get genuine reasoning or style artifacts.

---

**Tweet 6:**
Full explainer with DPO vs SimPO vs ORPO math,
real simulation output, and the complete diagnostic:

https://yakobdereje.substack.com/p/how-preference-training-actually

Sources:

- Meng et al. 2024 (SimPO) — arxiv.org/abs/2405.14734
- Gao et al. 2023 (Reward Overoptimization) — arxiv.org/abs/2210.10760
