# Morning Call Summary — Day 3 (Training and Post-Training Mechanics)

**Date:** Day 3
**Pair:** Yakob Dereje & Beamlak
**Prepared by:** Yakob Dereje | **Confirmed by:** Beamlak

**Yakob's original draft question (before the call):**

> "In my Week 11 Tenacious-Bench, I trained a LoRA adapter
> using SFT. My datasheet.md flags 94.3% augmented data as a
> limitation. What does cross-entropy loss actually optimize?"

Too broad — could go toward SFT mechanics generally, LoRA
rank choice, augmentation strategy, or evaluation validity.
Beamlak challenged: "Are you asking about what cross-entropy
measures token by token, or about whether your specific adapter
learned style vs memorization? Those need different explainers."

**Yakob's rewrite after the challenge:**
Named the exact mechanism (gradient flow through frozen W₀
into A and B), the exact artifact (datasheet.md limitation),
the exact number at stake (Delta A +0.263), and the exact
diagnostic consequence: can the adapter be defended as style
learning or not?

**Beamlak's original draft question (before the call):**

> "How do DPO, SimPO, and ORPO shift model behavior differently
> and which is best for reducing overcommitment?"

Too evaluative — "which is best" invites a recommendation not
a mechanism explanation. Yakob challenged: "Are you asking
which objective to use, or are you asking whether your
benchmark improvement reflects genuine reasoning or evaluator
gaming? The second is a much more defensible question."

**Beamlak's rewrite after the challenge:**
Named the exact mechanism (token policy shift), the exact
failure mode (evaluator gaming through style artifacts like
"scope", "validate", "phased approach"), and the exact
diagnostic need: a test that distinguishes genuine evidence-
conditional reasoning from surface token memorization.

Both questions committed final by end of call.
