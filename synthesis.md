# Week 12 Synthesis — Knowledge Gap Formulation for Compounding

**Yakob Dereje | TenX Academy TRP1 | Week 12**
**Date:** May 2026

---

## Overview

Week 12 asked one question of everything built in Weeks 0-11:
do you actually understand what you built, or were you following
patterns without internalizing the engineering principles beneath
them? Over four days of paired gap research, I closed ten gaps —
five I named from my own work, five I researched for partners.
Each gap produced a public artifact, a real edit to existing
portfolio work, and a deeper understanding of the systems I built.

---

## The Ten Gaps Closed

### Gaps I Named (Five I Asked)

**Gap 1 — Day 1: LoRA Inference Validity Boundary**
Topic: Inference-Time Mechanics

My Week 11 model card stated that Delta A (+0.263, p<0.0001)
was measured using system-prompt simulation rather than real
LoRA adapter weights. I could not explain whether this was a
valid proxy or an invalid one — I just flagged it as a
limitation without understanding the mechanism.

What I now know: System-prompt simulation and LoRA adapter
inference operate on fundamentally different parts of the
forward pass. The adapter modifies weight matrices
W_eff = W₀ + (α/r)·B@A; system-prompt simulation modifies
input activations while leaving weights unchanged. Merged and
unmerged LoRA inference are numerically identical to machine
precision (8.33e-17). My Delta A is valid for the four
rule-based rubric dimensions and an unvalidated proxy for the
tone-marker dimension (estimated ≤15% of overall score).

Grounding commit: Updated model_card.md Limitations section
with a technically grounded validity boundary statement.

---

**Gap 2 — Day 2: Reasoning Trace Token Cost**
Topic: Agent and Tool-Use Internals

My Week 10 mechanism run used qwen3-next-80b-a3b-thinking and
recorded additional_cost_usd: 0.0 in act4_mechanism.py. I
believed the mechanism added zero marginal cost. I had no idea
what reasoning trace tokens were or that thinking models charge
for them at output token rates.

What I now know: Every token generated inside the think block
goes through the same autoregressive decode loop as the visible
answer — one forward pass per token, full model weights loaded,
billed at output token rates. My confidence-aware abstention
prefix caused 96.9% reasoning tokens across 30 held-out
simulations — a 32:1 reasoning-to-answer ratio. The mechanism
adds no extra LLM calls but the deliberative prompt caused the
model to spend 32 tokens of internal reasoning for every 1
token of visible output.

Grounding commit: Added reasoning_token_note to
act4_mechanism.py metadata with the 97% ratio and audit
instructions.

---

**Gap 3 — Day 3: SFT Gradient Flow and Style vs Memorization**
Topic: Training and Post-Training Mechanics

My Week 11 datasheet.md flagged "94.3% augmented data from
128 originals" as a limitation — but I could not explain the
gradient-level mechanism that makes low diversity a real
problem. I knew the number. I could not defend what it meant.

What I now know: Cross-entropy loss measures next-token
prediction accuracy — not style directly. Near-duplicate
examples produce highly aligned gradient signals that push
LoRA A and B matrices in similar directions repeatedly. Loss
falls quickly on recurring token patterns — openers, CTA
phrasings, sentence shells — rather than generalizable style
policy. Two diagnostics distinguish style learning from
memorization: grouped holdout split (all augmentations from
one original must stay in the same split) and gradient norm
comparison by module (attention: q/k/v/o vs MLP:
gate/up/down).

Grounding commit: Updated datasheet.md Known Issues section
with a mechanically grounded explanation of the augmentation
diversity limitation and the two diagnostic methods.

---

**Gap 4 — Day 4: LLM Judge Bias Audit**
Topic: Evaluation and Statistics

My Week 11 judge achieved ≥80% inter-rater agreement. I
reported Delta A: +0.263 (p<0.0001) using this judge. I never
tested for position bias, length bias, or self-preference bias.
I could not answer whether my result reflected genuine quality
improvement or judge artifacts.

What I now know: Each bias manifests at the token level during
judge inference. Position bias: slot identity becomes a feature
before the judge has fully read the content. Length bias:
longer outputs contain more quality-signal tokens — rubric
keywords, caveats, connective phrases. Self-preference bias:
familiar text has lower perplexity for the judge, activating
quality-associated continuations. Cross-model judging partially
mitigates but does not eliminate self-preference. A live
position-swap audit (n=5) showed slot_A_win_rate jumping from
0.20 to 0.60 when outputs swapped positions — consistent with
position sensitivity. My Delta A is not invalid but is
under-audited.

Grounding commit: Added Judge Bias Audit Status section to
model_card.md Evaluation Results with conservative
interpretation and minimum check package.

---

### Gaps I Researched (Five I Explained)

**Gap 5 — Day 1: Preference Accuracy vs Rejection-Sampling Lift**
Partner: Nahom Desalegn

Nahom's system achieved 96.9% preference accuracy but near-zero
rejection-sampling lift (Delta A ≈ 0, 17/52 ties). He could not
explain the structural reason for this divergence.

What I researched and taught: The margin distribution mismatch
mechanism. Training pairs are deliberately different — large
quality margin. Deployment candidates are generated by the same
model at the same temperature — low variance, small margin. A
simulation demonstrated the gap: training pairs were 12.5x more
discriminable than deployment candidates. The cheap diagnostic:
measure score_std of deployment candidate pool. If score_std
< 0.1, the problem is generation not the critic.

---

**Gap 6 — Day 2: Function Calling at the Token Level**
Partner: Yosef Zewdu

Yosef's LangGraph agent routes on whether tool_calls exists in
the model's response. He could not explain what the model
actually generates to create that field.

What I researched and taught: Tool calling is token prediction
not decision making. Tool schemas from tools.py are injected
into the prompt as plain text. The model generates structured
JSON tokens — {"name": "hubspot_upsert_contact", "arguments":
{...}} — which OpenRouter intercepts, parses, and converts into
the tool_calls object the router reads. A live demonstration
using the OpenRouter API produced real tool_calls output:
finish_reason=tool_calls, tool name correctly parsed. Tools are
ignored when descriptions are vague or requests are ambiguous.

---

**Gap 7 — Day 3: DPO vs SimPO vs ORPO Token Policy Shift**
Partner: Beamlak

Beamlak's benchmark improved after preference training but she
could not defend whether the improvement reflected genuine
evidence-aware reasoning or evaluator gaming through style
artifacts like "scope", "validate", "phased approach."

What I researched and taught: Each objective shifts token
probability distributions differently. DPO anchors to a
reference model — medium gaming risk. SimPO removes the
reference model, normalizes by length — higher gaming risk.
ORPO combines SFT and preference — lower gaming risk. A
simulation showed genuine learning raises evidence and discovery
tokens most; evaluator gaming raises scope and validate most.
The diagnostic: ratio of cautious tokens in weak evidence vs
strong evidence responses. Ratio > 2.0 indicates genuine
reasoning.

---

**Gap 8 — Day 4: Paired Bootstrap Null Distribution Construction**
Partner: Rafia Kedir

Rafia ran a paired bootstrap test, got p=0.189, recorded
DO NOT DEPLOY — but could not explain why paired bootstrap was
correct, what p=0.189 means mathematically, or how her 60/40
blended scoring formula affected test sensitivity.

What I researched and taught: Paired bootstrap is correct for
NLP evaluation because it makes no distributional assumptions —
unlike t-test which assumes normality. The null distribution is
constructed by resampling score differences with replacement
10,000 times and shifting to center at zero. p=0.189 means
18.9% of bootstrap resamples showed a delta as large as
observed under the null — not significant at alpha=0.05, but
not evidence of zero effect. The 60/40 blending reduced score
variance by 36%, creating asymmetric comparison between
baseline (pure machine scores) and trained judge (blended
scores). With n=59 tasks, approximately 150-200 tasks are
needed to reach p<0.05 at the observed effect size.

---

## The Most Surprising Thing I Learned

The most surprising insight from the week was from Day 2 —
that my Week 10 "9-step automated agent" does not use LLM
function calling at all. Every tool call to HubSpot, Cal.com,
Resend, and Africa's Talking is made directly by Python.
The model only generates text. My memo explicitly states
"Enrichment, composition, and tone-checking are all rule-based
with no LLM calls" — yet I described the system as an agent
that "routes prospects" and "calls tools."

This forced me to understand the actual distinction between
a true LLM function-calling agent (where the model generates
structured JSON that a runtime intercepts and executes) and a
Python-orchestrated pipeline (where code makes all decisions
and the model generates text). The distinction matters for
debugging, cost estimation, architecture decisions, and how
you describe the system to clients.

---

## Canonical Reading List

See canonical_list.md for the full annotated list.

**Papers:**
- Hu et al. 2022 — LoRA (arXiv:2106.09685)
- Meng et al. 2024 — SimPO (arXiv:2405.14734)
- Gao et al. 2023 — Reward Overoptimization (arXiv:2210.10760)
- Dror et al. 2018 — Statistical Significance in NLP (ACL 2018)
- Schick et al. 2023 — Toolformer (arXiv:2302.04761)
- Patil et al. 2023 — Gorilla (arXiv:2305.15334)
- Stiennon et al. 2020 — RLHF Summarization (arXiv:2009.01325)
- Lambert et al. 2024 — RewardBench (arXiv:2403.13787)
- Efron & Tibshirani 1993 — An Introduction to the Bootstrap
- Zheng et al. 2023 — MT-Bench (arXiv:2306.05685)

**Tools:**
- numpy — simulation and statistical verification
- PEFT / HuggingFace — LoRA adapter training and inference
- OpenRouter — LLM routing with tool calling support
- Unsloth — efficient LoRA fine-tuning on Colab T4
- GroupShuffleSplit (sklearn) — grouped holdout validation

**Patterns:**
- Position-swap audit — cheapest LLM judge bias diagnostic
- Margin distribution diagnostic — score_std of deployment pool
- Grouped holdout split — style vs memorization validation
- Gradient norm by module — attention vs MLP training pressure
- Paired bootstrap — correct significance test for NLP benchmarks

---

## What Week 12 Changed

At the start of Week 12 I had four artifacts I could ship but
not fully defend. By the end I can explain the forward pass
mechanics of my LoRA adapter, the token-level behavior of my
reasoning model, the gradient dynamics of my SFT training, and
the bias profile of my LLM judge. Each understanding produced
a concrete edit to existing work — not a theoretical note but
an actual improvement to what a hiring manager or senior
engineer would read.

The week confirmed the program's core claim: shipping systems
is necessary but not sufficient. The ability to explain, defend,
and teach what you built is what separates an engineer who can
execute from one who can lead.