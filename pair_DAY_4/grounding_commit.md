# Grounding Commit — Day 4

**Asker:** Yakob Dereje
**Repository edited:** yakobd/The_Sales_Agent_Evaluation_Bench
**File edited:** model_card.md
**Section:** Evaluation — Inter-Rater Agreement

## Before (original text)

> "Inter-rater agreement was computed on 30 tasks labeled
> twice 24 hours apart. Agreement exceeded 80% on all 5
> rubric dimensions (overall 93%). No rubric revision was
> required."

## After (replacement text)

> **Inter-rater agreement:** Computed on 30 tasks labeled
> twice 24 hours apart. Agreement exceeded 80% on all 5
> rubric dimensions (overall 93%). No rubric revision was
> required.
>
> **Bias audit status:** The judge was NOT tested for the
> three known LLM-as-a-judge biases before reporting Delta A.
>
> Position bias: Unknown. Output presentation order was not
> randomized during evaluation. A position-swap audit (re-judge
> 20+ pairs with order reversed, measure flip rate) has not
> been run. Red flag threshold: slot_A_win_rate > 0.65 in
> both orderings, or flip_rate > 0.35.
>
> Length bias: Partially screened. A length-score correlation
> check on existing deterministic artifacts showed negative
> correlation between output length and score (Pearson -0.51
> for trained outputs) — no obvious positive length bias in
> saved deterministic artifacts. The live LLM judge has not
> been audited separately.
>
> Self-preference bias: Partially mitigated. Generator and
> judge used different Qwen model families (generation:
> Qwen2.5-72B, judge: Qwen3-series). This prevents direct
> self-grading but does not eliminate shared post-training
> style familiarity bias. A lineage stratification audit
> has not been run.
>
> **Conservative interpretation:** Delta A = +0.263
> (p<0.0001) is a strong result under the current judge
> protocol. Because the judge was not fully audited for
> position, length, or self-preference bias, systematic
> judge bias could explain part of the lift. The result
> should be read as: improvement under current judge
> protocol, pending full bias audit.

## Why This Changed

Before Day 4 I could not explain what the three LLM judge
biases were, how they manifest at the token level, or
whether my mitigation strategy (cross-model judging) was
sufficient. I knew there were biases but had no language
to describe their boundary precisely.

After reading Nebiyou's explainer I understand that each
bias operates at the token level during judge inference,
that cross-model judging is partial mitigation not full
clearance, and that my 80% agreement does not address
presentation-order or verbosity sensitivity. The original
model card implied the judge was validated — the new version
is honest about what was and was not audited.
