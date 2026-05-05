# Sources — Day 1

**Explainer written by:** Yakob Dereje
**For question asked by:** Nahom Desalegn

---

## Canonical Paper 1

**Stiennon et al. (2020) — "Learning to summarize with human feedback"**
NeurIPS 2020
https://arxiv.org/abs/2009.01325

Introduces rejection sampling (Best-of-N) with reward models in a
production context. Documents the full pipeline of reward model
training and policy optimization. Load-bearing for the mechanism
section explaining why preference accuracy and deployment lift
decouple.

---

## Canonical Paper 2

**Lambert et al. (2024) — "RewardBench: Evaluating Reward Models
for Language Modeling"**
arXiv:2403.13787
https://arxiv.org/abs/2403.13787

Systematic evaluation of reward model behavior across chat,
reasoning, and safety domains. Directly relevant to the divergence
between preference accuracy on structured pairs and downstream
deployment performance. Load-bearing for the diagnostic and
decision-rule sections.

---

## Tool / Pattern Used

**Margin distribution simulation — Python (numpy, seed=42)**

Simulated 1,000 training pairs (high margin, mean=1.808) against
100 deployment candidate pools of k=8 (low margin, mean=0.145).
Demonstrated 12.5x discriminability gap between training
distribution and deployment distribution. All output verified
by running the code locally. Full simulation in explainer.md.

---

## Follow-On Reading

**Gao et al. (2023) — "Scaling Laws for Reward Model
Overoptimization"**
arXiv:2210.10760
What happens when you keep selecting highest-reward outputs over
many iterations. The next failure mode to understand after the
margin distribution problem is solved.
