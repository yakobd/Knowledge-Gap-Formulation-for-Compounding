# Grounding Commit — Day 3

**Asker:** Yakob Dereje
**Repository edited:** yakobd/The_Sales_Agent_Evaluation_Bench
**File edited:** datasheet.md
**Section:** Limitations — Augmentation-based training data

## Before (original text)

> "Augmentation-based training data — 94.3% of training pairs
> are augmented variations of 128 originals. Diversity is lower
> than 2,541 independently authored pairs would provide."

## After (replacement text)

> **Augmentation-based training data:** 94.3% of training pairs
> are augmented variations of 128 originals. Diversity is lower
> than 2,541 independently authored pairs would provide.
>
> **Gradient-level mechanism:** Near-duplicate examples produce
> highly aligned gradient signals during SFT. Repeated aligned
> gradients push LoRA A and B matrices in similar directions
> across training steps, causing loss to fall quickly on
> recurring token patterns — openers, CTA phrasings, sentence
> shells — rather than generalizable style policy. This can
> produce real metric gains (Delta A: +0.263, p<0.0001) while
> the adapter captures surface regularities rather than
> generalizable Tenacious tone.
>
> **Diagnostic:** To distinguish style learning from
> augmentation-family memorization, apply a grouped holdout
> split where all augmentations derived from one original email
> stay in the same split. Sharp performance drop under grouped
> holdout vs standard holdout indicates memorization. Gradient
> norm comparison by module (attention: q/k/v/o vs MLP:
> gate/up/down) identifies whether training pressure concentrated
> in phrase-shaping layers vulnerable to surface memorization.
>
> **Conservative interpretation:** Delta A should be read as
> improvement on the measured evaluation distribution. Whether
> it reflects generalizable style learning or augmentation-family
> memorization requires grouped holdout validation not yet run.

## Why This Changed

Before Day 3 I could not explain why low diversity is a gradient-
level problem — only that it was a limitation. I knew the 94.3%
number but could not connect it to what happens during training.

After reading Beamlak's explainer I understand that near-duplicate
examples create aligned gradients that repeatedly push A and B
in the same directions — causing surface pattern reinforcement
that masquerades as style learning. The original limitation was
honest but vague. The new version explains the mechanism and
gives two concrete diagnostics for validating the claim.
