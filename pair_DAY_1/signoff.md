# Sign-off — Day 1

**Asker:** Yakob Dereje
**Explainer written by:** Nahom Desalegn
**Gap verdict:** CLOSED ✅

## What I Now Understand That I Did Not Before

**Before today:**
I knew my model card had a limitation — evaluation used system-prompt
simulation rather than real LoRA weights — but I could not explain
WHY that mattered mechanically, or HOW MUCH it mattered for my
specific rubric. The limitation statement in my model card was honest
but vague. I could not defend it if a senior engineer pushed back.

**After today, I understand three things precisely:**

**1. The mechanism difference is fundamental, not cosmetic.**
The adapter modifies weight matrices: W_eff = W₀ + (α/r)·B@A.
System-prompt simulation modifies input activations while leaving
weights unchanged. These operate on different parts of the
computation graph. They can produce similar behavioral outputs on
string-matching tasks without producing similar token-probability
distributions.

**2. Merged and unmerged inference are numerically identical.**
Maximum absolute difference = 8.33e-17 (floating-point noise only).
This was never a concern — I just did not know that before today.
The choice between merged and unmerged at serving time is purely
a performance decision, not a correctness one.

**3. My Delta A result has a precise, defensible validity boundary.**
- For 4 of 5 rubric dimensions (banned phrases, ICP fingerprints,
  timezone tokens, honesty flags): system-prompt simulation is a
  VALID proxy. These dimensions are string-sensitive, not
  distribution-sensitive. They cannot see the mechanism.
- For the tone-marker dimension (LLM sub-judge): system-prompt
  simulation is an UNVALIDATED proxy. Fine-grained distributional
  differences may produce divergent scores here.
- Delta A (+0.263) is valid for the four rule-based dimensions and
  an upper bound of unknown tightness for tone. This is defensible.
  It just needs to be stated precisely — which it now is.

## What Changed in My Existing Work

Updated model_card.md Limitations section in
yakobd/The_Sales_Agent_Evaluation_Bench with a technically grounded
replacement that names the validity boundary precisely.
See grounding_commit.md for the full diff and explanation.
