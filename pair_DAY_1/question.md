# Day 1 Question — Inference-Time Mechanics

**Asked by:** Yakob Dereje
**Date:** Day 1
**Topic:** Inference-Time Mechanics

## The Question

When a LoRA adapter is loaded for inference, how do the adapter
matrices A and B actually modify the transformer forward pass —
and does the difference between merged adapter inference, unmerged
adapter inference, and system-prompt simulation produce meaningfully
different output token distributions — and does that difference mean
my Delta A result of +0.263 requires a validity caveat in my
model card?

## Artifact Pointer

This gap lives in `model_card.md` of my Week 11 repository
(`yakobd/The_Sales_Agent_Evaluation_Bench`) under the Limitations
section, where I documented:

> "ablation scoring used the trained system prompt via API rather
> than loading LoRA weights directly. The adapter was trained and
> saved correctly but direct weight loading was not tested at
> evaluation time."

My headline result — **Delta A: +0.263, p<0.0001** — was measured
using system prompt simulation, not real adapter weights. I cannot
currently defend whether that result holds under real LoRA inference.

## Why This Gap Matters

Closing this gap would let me either:

1. Confidently defend my Delta A result as a valid proxy for real
   adapter performance, or
2. Add a concrete, technically grounded correction to my model card
   explaining the validity boundary and what a proper evaluation
   would require.

Any FDE deploying a fine-tuned adapter — whether for email
generation, code review, or document summarization — and evaluating
it via API proxy rather than direct weight loading faces this same
validity question. The gap is not specific to B2B email; it applies
to any system where training and evaluation use different inference
paths.

## What a Satisfying Answer Looks Like

A good explainer would:

- Explain exactly what changes in the transformer forward pass when
  LoRA weights are loaded (merged vs unmerged)
- Show whether system-prompt simulation is a valid proxy or not
- Include runnable code or concrete output demonstrating the difference
- Tell me whether my Delta A result is defensible or needs a caveat
