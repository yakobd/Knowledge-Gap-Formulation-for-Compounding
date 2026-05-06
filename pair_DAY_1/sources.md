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

RewardBench evaluates reward models across chat, reasoning, and
safety categories using prompt-chosen-rejected trios. Its core
finding is that reward models achieving high accuracy on chat-style
pairs frequently underperform on reasoning and safety categories —
demonstrating that accuracy on the training distribution does not
transfer reliably to deployment distributions. This cross-category
accuracy collapse is the empirical foundation for the diagnostic
argument in the explainer: high preference accuracy is a property
of the training distribution, not a guarantee of deployment lift.
Load-bearing for the diagnostic and decision-rule sections.

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
