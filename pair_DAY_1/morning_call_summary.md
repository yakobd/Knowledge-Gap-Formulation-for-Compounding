# Morning Call Summary — Day 1 (Inference-Time Mechanics)

**Date:** May 5, 2026
**Pair:** Yakob Dereje & Nahom Desalegn
**Duration:** ~32 minutes
**Prepared by:** Nahom Desalegn | **Confirmed by:** Yakob Dereje

## What Was Ambiguous in the Original Drafts

**Yakob's original draft question (before the call):**

> "When a LoRA adapter is loaded for inference, what actually happens
> inside the model — and does it matter that I used a system prompt
> instead of loading the adapter weights during my evaluation?"

This was too broad. It could have gone in five directions — the
training mechanics of LoRA, the PEFT library implementation, the
inference speed difference, the cost difference, or the evaluation
validity question. A peer receiving this could not know which
direction to research.

**Nahom's challenge during the call:**
"Are you asking about the mechanism of the forward pass itself —
how A and B modify the computation — or are you asking whether
your specific evaluation results are valid given that you used a
system prompt instead of real weights? Those are two different
questions. Which one would actually change something in your
existing work?"

**Yakob's rewrite after the challenge:**

> "When a LoRA adapter is loaded for inference, how do the adapter
> matrices A and B actually modify the transformer forward pass —
> and does the difference between merged adapter inference, unmerged
> adapter inference, and system-prompt simulation produce meaningfully
> different output token distributions — and does that difference
> mean my Delta A result of +0.263 requires a validity caveat in
> my model card?"

The rewrite names the exact mechanism (A and B matrices, three
inference modes), the exact number at stake (Delta A +0.263), and
the exact artifact that needs improving (model_card.md Limitations).
It collapsed five possible directions into one precise question.

---

**Nahom's original draft question (before the call):**

> "Why does high preference accuracy not predict rejection-sampling
> lift, and what should I do about it?"

Too general — no numbers, no artifact pointer, no specific mechanism
named. Reads like a theoretical curiosity rather than an engineering
problem.

**Yakob's challenge during the call:**
"What are the actual numbers from your work? What file would you
edit if you understood this? Is the problem in your training setup
or your deployment setup — or do you not know which?"

**Nahom's rewrite after the challenge:**
Named exact numbers: 96.9% preference accuracy on validation pairs
vs Delta A = +0.0025 (p=0.40) with 17/52 ties on held-out tasks.
Grounded to specific artifacts: ablations/ablation_results.json,
tenacious_bench_v0.1 dataset. Core puzzle made explicit: the
structural relationship between training-pair margin distribution
and inference-time candidate pool variance — and the decision it
needs to resolve: training-side fix vs deployment-side fix.

## Outcome

Both questions finalized by end of call. Each question is
unambiguous to the other partner. Questions committed final.
