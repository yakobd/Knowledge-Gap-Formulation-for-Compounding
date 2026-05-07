# Sign-off — Day 3

**Asker:** Yakob Dereje
**Explainer written by:** Beamlak
**Gap verdict:** CLOSED ✅

## What I Now Understand That I Did Not Before

**Before today:**
I reported Delta A: +0.263 (p<0.0001) and flagged "94.3%
augmented data" as a limitation in my datasheet.md — but I
could not explain the gradient-level mechanism that makes low
diversity a real problem. I knew the number. I could not defend
what it meant for what the adapter actually learned.

**After today, I understand three things precisely:**

**1. Cross-entropy measures next-token prediction — not style.**
The loss scores how surprised the model was by each gold token.
Style enters indirectly because my training targets contain
Tenacious-style outputs. But the loss never explicitly optimizes
for "Tenacious voice" as an abstract concept — only for local
next-token accuracy at each position.

**2. Low diversity creates aligned gradients that reward surface
patterns.**
94.3% augmented pairs from 128 originals means near-duplicate
examples produce highly aligned gradient signals. Repeated
aligned gradients push A and B in similar directions repeatedly.
Loss falls quickly on recurring token patterns — openers, CTA
phrasings, sentence shells. This looks like strong learning
even when the adapter captured surface regularities, not style.

**3. Two concrete diagnostics distinguish style from memorization.**
Grouped holdout split: all augmentations from one original must
stay in the same split. If performance drops sharply under
grouped holdout but stays high on standard holdout — gains
depended on augmentation-family overlap, not generalization.
Gradient norm by module: attention-side updates indicate
context-to-decision routing; MLP-side updates indicate lexical
transformation vulnerable to phrase memorization.

## What Changed in My Existing Work

Updated datasheet.md Limitations section in
yakobd/The_Sales_Agent_Evaluation_Bench to replace the vague
augmentation flag with a mechanically grounded explanation.
See grounding_commit.md for full details.
