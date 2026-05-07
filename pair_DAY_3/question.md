# Day 3 Question — Training and Post-Training Mechanics

**Asked by:** Yakob Dereje
**Date:** Day 3
**Topic:** Training and Post-Training Mechanics

## The Question

In my Week 11 Tenacious-Bench, I trained a LoRA adapter on
Tenacious-style B2B sales emails using Supervised Fine-Tuning
(SFT). My training script minimizes cross-entropy loss — but
I cannot explain what that means at the token level, which of
my seven target weight matrices (q_proj, k_proj, v_proj, o_proj,
gate_proj, up_proj, down_proj) received the largest gradient
updates, or whether the adapter learned generalizable Tenacious
tone style versus surface-pattern memorization from a training
set that is 94.3% augmented variations of only 128 originals.

When SFT cross-entropy loss backpropagates through a LoRA adapter
— what is measured token by token, how do gradients flow through
frozen W₀ into only A and B matrices, and what does the resulting
weight shift mean for whether my adapter learned how Tenacious
writes versus what Tenacious wrote?

## Artifact Pointer

- `datasheet.md` in `yakobd/The_Sales_Agent_Evaluation_Bench`
  — flags "94.3% of training pairs are augmented variations of
  128 originals. Diversity is lower than 2,541 independently
  authored pairs would provide" — a limitation I stated but
  cannot mechanically explain at the gradient level
- `training/` directory — training script optimizing
  cross-entropy loss on Tenacious email pairs
- Delta A: +0.263 (p<0.0001) — a result I can report but
  cannot defend in terms of what the loss function actually
  optimized

## Why This Gap Matters

Closing this gap would let me rewrite the Limitations section
of my datasheet.md with a mechanically grounded explanation
of why low diversity is a real problem — not just a flag.

Any FDE who trains a LoRA adapter using SFT on a narrow
augmented dataset — for email composition, code generation,
document summarization, or any domain-specific task — faces
this exact question. The gap between "the loss converged" and
"the adapter learned what I intended" is one of the most
common undefended claims in production fine-tuning.

## What a Satisfying Answer Looks Like

A good explainer would:

- Walk through what cross-entropy loss measures token by token
- Show how gradients flow through frozen W₀ into A and B
- Explain which weight matrices are most likely to encode
  style vs content
- Give a concrete diagnostic that distinguishes a style-learning
  adapter from a memorization adapter
- Tell me whether my Delta A result is more likely genuine
  style learning or augmentation memorization
