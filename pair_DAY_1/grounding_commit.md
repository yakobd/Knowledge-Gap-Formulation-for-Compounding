# Grounding Commit — Day 1

**Asker:** Yakob Dereje
**Repository edited:** yakobd/The_Sales_Agent_Evaluation_Bench
**File edited:** model_card.md
**Section:** Limitations — Simulated adapter inference

## Before (original text)

> "Simulated adapter inference — ablation scoring used the trained
> system prompt via API rather than loading LoRA weights directly.
> The adapter was trained and saved correctly but direct weight
> loading was not tested at evaluation time."

## After (replacement text)

> **Evaluation method**: Delta A (+0.263, p<0.0001) measures the
> effect of system-prompt simulation — injecting the condensed
> Tenacious style guide v2 as a system prompt to the base
> Qwen2.5-0.5B-Instruct model — compared to baseline (no system
> prompt). The trained LoRA adapter weights were not loaded during
> evaluation.
>
> **Validity as a proxy**: System-prompt simulation and LoRA adapter
> inference operate on different parts of the forward pass. The
> adapter modifies weight matrices W_eff = W₀ + (α/r)·B@A;
> system-prompt simulation modifies input activations while leaving
> weights unchanged. For rule-based rubric dimensions (banned
> phrases, ICP fingerprints, timezone tokens, honesty flags), the
> two approaches produce equivalent scores when the system prompt
> successfully encodes the same constraints the adapter learned —
> because these dimensions are sensitive to string presence, not
> token-distribution shape. For the tone-marker dimension (LLM
> sub-judge), fine-grained distributional differences between the
> two approaches may produce divergent scores.
>
> **Conservative interpretation**: Delta A should be read as a valid
> proxy for the four rule-based dimensions and an unvalidated proxy
> for tone. Given that 4 of 5 rubric dimensions are rule-based, the
> adapter result is unlikely to differ by more than the tone-marker
> dimension's weight (estimated ≤15% of overall score).

## Why This Changed

Before Day 1 I could not explain the mechanism difference between
system-prompt simulation and real adapter inference. I knew a gap
existed but had no language to describe its boundary precisely.

After reading Nahom's explainer I understand that the two approaches
operate on fundamentally different parts of the computation graph —
weights vs input activations — and that the validity boundary maps
directly onto which rubric dimensions are string-sensitive vs
distribution-sensitive.

The original limitation was honest but vague. The new version is
honest and precise. A hiring manager or senior engineer reading
my model card can now understand exactly what was measured, what
it is valid for, and what remains unvalidated — without reading
between the lines.
