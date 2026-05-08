# Day 4 Question — Evaluation and Statistics

**Asked by:** Yakob Dereje
**Date:** Day 4
**Topic:** Evaluation and Statistics

## The Question

In my Week 11 Tenacious-Bench, my LLM judge achieved ≥80%
inter-rater agreement — but I never tested for position bias,
length bias, or self-preference bias. My Delta A: +0.263
(p<0.0001) was measured using this judge.

I cannot defend this result against three specific unknowns:
I don't know whether baseline and mechanism outputs were
presented in randomized order (position bias risk), I don't
know whether mechanism outputs were systematically longer
than baseline (length bias risk), and I can't confirm whether
using different Qwen model families for generation and judging
eliminates or only partially mitigates self-preference bias.

**Specific gap:** How does each bias manifest at the token
level during judge inference, what is the cheapest diagnostic
for each, and does cross-model judging (different Qwen families
for generation vs judging) eliminate or only partially mitigate
self-preference bias?

## Artifact Pointer

- `model_card.md` in `yakobd/The_Sales_Agent_Evaluation_Bench`
  — inter-rater agreement section reporting ≥80% agreement
  with no mention of position, length, or self-preference
  bias testing
- `ablation_results.json` — Delta A: +0.263 (p<0.0001) —
  a result whose validity depends on whether the judge
  is unbiased
- I do not know whether my evaluation randomized presentation
  order or whether mechanism outputs were systematically
  longer than baseline outputs

## Why This Gap Matters

Closing this gap would let me add a technically grounded
bias testing section to my model card — replacing the current
inter-rater agreement claim with a defensible statement about
what bias testing was and was not done.

Any FDE who uses an LLM as a judge — for email scoring, code
review, proposal ranking, meeting summary grading, or retrieval
reranking — and reports results without testing for these three
biases is producing potentially inflated metrics. The gap
applies to every LLM-as-a-judge system regardless of domain.

## What a Satisfying Answer Looks Like

A good explainer would:

- Name each bias mechanically — how it manifests at the
  token level during judge inference
- Give the cheapest observable diagnostic for each bias
- Explain whether cross-model judging eliminates or only
  partially mitigates self-preference bias
- Tell me whether my Delta A result is defensible given
  what I know about my evaluation setup
