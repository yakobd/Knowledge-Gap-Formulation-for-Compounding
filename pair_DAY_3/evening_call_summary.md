# Evening Call Summary — Day 3 (Training and Post-Training Mechanics)

**Date:** Day 3
**Pair:** Yakob Dereje & Beamlak
**Prepared by:** Yakob Dereje | **Confirmed by:** Beamlak

Yakob's explainer on preference training token policy shift was
revised to strengthen the connection between the simulation
numbers and Beamlak's specific benchmark — adding the concrete
probe diagnostic with strong evidence vs weak evidence prompts
and making the decision rule (ratio greater than 2.0) more
actionable for her specific bench_overcommitment failure mode.

Beamlak's explainer on SFT cross-entropy gradient flow was
revised to add the grouped holdout split code and gradient norm
logging sketch — replacing a theoretical description with a
minimal runnable check that Yakob can apply directly to his
Tenacious-Bench training setup.

Yakob confirmed Beamlak's explainer closed his gap — the
distinction between "loss converged" and "adapter learned
generalizable style" is now mechanically grounded through the
grouped holdout and gradient norm diagnostics.

Beamlak confirmed Yakob's explainer closed her gap — the
difference between genuine evidence-conditional reasoning and
evaluator gaming is now visible through the token probability
simulation and the ratio diagnostic.

Both gaps fully closed ✅
