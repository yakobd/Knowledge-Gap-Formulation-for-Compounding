# Morning Call Summary — Day 1 (Inference-Time Mechanics)

**Date:** May 5, 2026
**Pair:** Yakob Dereje & Nahom Desalegn
**Duration:** ~32 minutes
**Prepared by:** Nahom Desalegn | **Confirmed by:** Yakob Dereje

## What Was Ambiguous in the Original Drafts

**Yakob's question (LoRA inference):**
Original draft asked broadly "what happens during LoRA inference."
Nahom pushed back during the call: "Are you asking about the
mechanism of the forward pass, or about whether your specific
evaluation results are valid?" The question was too wide to produce
a focused explainer — it could have gone in five directions.

**Nahom's question (preference accuracy vs lift):**
Original draft was general about the divergence between training
and deployment metrics. No specific numbers were named. It read
like a theoretical curiosity rather than a grounded engineering
problem.

## How Each Question Was Sharpened

**Yakob's question after sharpening:**
Rewritten to name the exact artifact (model_card.md Limitations
section), the exact number at stake (Delta A: +0.263, p<0.0001),
and the exact validity question: does system-prompt simulation
produce equivalent token distributions to real adapter inference,
and where is the validity boundary?

**Nahom's question after sharpening:**
Rewritten to name exact numbers from Nahom's work: 96.9% preference
accuracy on validation pairs vs Delta A = +0.0025 (p=0.40) with
17/52 ties on held-out tasks. Grounded to specific artifacts
(ablations/ablation_results.json, tenacious_bench_v0.1 dataset).
Core puzzle made explicit: the structural relationship between
training-pair margin distribution and inference-time candidate pool
variance — and the decision it needs to resolve: training-side fix
vs deployment-side fix.

## Outcome

Both questions finalized by end of call. Each question is
unambiguous to the other partner. Questions committed final.
