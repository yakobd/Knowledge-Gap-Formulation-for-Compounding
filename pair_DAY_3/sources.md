# Sources — Day 3

**Explainer written by:** Yakob Dereje
**For question asked by:** Beamlak

---

## Canonical Paper 1

**Meng et al. (2024) — "SimPO: Simple Preference Optimization
with a Reference-Free Reward"**
arXiv:2405.14734
https://arxiv.org/abs/2405.14734

Defines the length-normalized reward formulation and compares
SimPO to DPO mathematically. Section 3 explains exactly how
removing the reference model changes the token probability
shift — load-bearing for the DPO vs SimPO vs ORPO mechanism
section and the evaluator gaming risk comparison.

---

## Canonical Paper 2

**Gao et al. (2023) — "Scaling Laws for Reward Model
Overoptimization"**
arXiv:2210.10760
https://arxiv.org/abs/2210.10760

Empirically demonstrates how preference-trained models overfit
to reward signals as training progresses. The foundational
paper on reward overoptimization and evaluator gaming.
Load-bearing for the diagnostic section and the adjacent
concept on KL regularization.

---

## Tool / Pattern Used

**Token probability shift simulation — Python (numpy, seed=42)**

Simulated token probability distributions before training,
after genuine reasoning learning, and after evaluator gaming.
Demonstrated that in genuine learning evidence and discovery
tokens rise more than scope and validate — while in evaluator
gaming scope and validate rise dramatically regardless of
evidence quality. All output verified by running the code
locally. Full simulation in explainer.md.

---

## Follow-On Reading

**Rafailov et al. (2023) — "Direct Preference Optimization:
Your Language Model is Secretly a Reward Model"**
arXiv:2305.18290
The original DPO paper — foundational for understanding how
preference objectives shift token policy relative to a reference
model. Essential next read after SimPO for anyone comparing
preference training objectives.
