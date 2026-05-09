# Canonical List — Week 12 Contribution to Cohort Canon

**Yakob Dereje | TenX Academy TRP1 | Week 12**

This is my annotated contribution to the cohort's shared
knowledge base — papers, tools, and patterns worth reading
for any Forward-Deployed Engineer working with LLM systems.

---

## Papers

---

**Hu et al. (2022) — "LoRA: Low-Rank Adaptation of Large
Language Models"**
arXiv:2106.09685
https://arxiv.org/abs/2106.09685

Why it matters: The foundational paper for fine-tuning large
models cheaply. Section 4 defines W_eff = W₀ + (α/r)·B@A,
the zero-initialization of B, and the alpha/r scaling. Section
7 proves merged and unmerged inference are numerically
identical. Essential reading before training any adapter or
reporting evaluation results from adapter inference. Every FDE
who fine-tunes a model without reading this is making
undefended architectural choices.

---

**Meng et al. (2024) — "SimPO: Simple Preference Optimization
with a Reference-Free Reward"**
arXiv:2405.14734
https://arxiv.org/abs/2405.14734

Why it matters: Defines length-normalized reward as the core
innovation over DPO. Section 3 explains exactly how removing
the reference model changes token probability shift — and why
this increases evaluator gaming risk. Read alongside the DPO
paper to understand what each objective actually optimizes at
the gradient level. Critical for any FDE who reports benchmark
improvements from preference training.

---

**Gao et al. (2023) — "Scaling Laws for Reward Model
Overoptimization"**
arXiv:2210.10760
https://arxiv.org/abs/2210.10760

Why it matters: Empirically demonstrates how preference-trained
models overfit to reward signals as training progresses. The
foundational paper on reward overoptimization and evaluator
gaming. Shows that benchmark improvement and real quality
improvement decouple at scale. Essential reading before
reporting any preference training result.

---

**Dror et al. (2018) — "The Hitchhiker's Guide to Testing
Statistical Significance in Natural Language Processing"**
ACL 2018
https://aclanthology.org/P18-1128/

Why it matters: The canonical reference for choosing the right
statistical test in NLP evaluation. Shows why t-tests fail for
non-normal evaluation measures, when paired bootstrap is
correct, and how most ACL papers misuse significance testing.
Every FDE who reports a p-value without reading this is at
risk of using the wrong test. Includes a practical protocol
for test selection.

---

**Schick et al. (2023) — "Toolformer: Language Models Can
Teach Themselves to Use Tools"**
arXiv:2302.04761
https://arxiv.org/abs/2302.04761

Why it matters: The foundational paper showing how models
learn to generate structured tool call tokens through training.
Section 3 explains the token-level mechanism — how API calls
are represented as text in training data. Essential for
understanding why tool calling is token prediction not decision
making, and why tool description quality determines reliability.

---

**Patil et al. (2023) — "Gorilla: Large Language Model
Connected with Massive APIs"**
arXiv:2305.15334
https://arxiv.org/abs/2305.15334

Why it matters: Demonstrates that tool calling is a token
generation problem — models must produce syntactically and
semantically correct structured output matching API schemas.
Shows why models hallucinate wrong tool arguments when
descriptions are vague. Directly load-bearing for tool
description engineering as a reliability lever.

---

**Stiennon et al. (2020) — "Learning to Summarize with
Human Feedback"**
NeurIPS 2020
https://arxiv.org/abs/2009.01325

Why it matters: Introduces rejection sampling (Best-of-N)
with reward models in a production context. The foundational
paper for understanding why preference accuracy and deployment
lift decouple — training pairs are high-margin by construction
while deployment candidates are low-margin by physics.
Essential for any FDE building a critic-gated system.

---

**Lambert et al. (2024) — "RewardBench: Evaluating Reward
Models for Language Modeling"**
arXiv:2403.13787
https://arxiv.org/abs/2403.13787

Why it matters: Systematic evaluation of reward model behavior
across chat, reasoning, and safety domains. Core finding:
reward models achieving high accuracy on chat-style pairs
frequently underperform on reasoning and safety categories.
Empirical proof that accuracy on the training distribution
does not transfer reliably to deployment distributions.

---

**Efron and Tibshirani (1993) — "An Introduction to
the Bootstrap"**
Chapman and Hall/CRC
ISBN 0-412-08231-2

Why it matters: The foundational textbook defining bootstrap
resampling, null distribution construction, and p-value
computation. Chapter 16 covers hypothesis testing via
bootstrap. Essential background before using or interpreting
any bootstrap confidence interval or p-value in an NLP
evaluation. Still the definitive reference 30 years later.

---

**Zheng et al. (2023) — "Judging LLM-as-a-Judge with
MT-Bench and Chatbot Arena"**
arXiv:2306.05685
https://arxiv.org/abs/2306.05685

Why it matters: The foundational paper on LLM-as-a-judge
evaluation. Identifies position bias, verbosity bias, and
self-enhancement bias as the three systematic failure modes.
Shows that strong judges can reach 80%+ agreement with humans
while still exhibiting these biases. Essential reading before
reporting any result measured by an LLM judge.

---

## Tools

---

**numpy (Python)**
https://numpy.org

Why it matters: The foundation for any statistical simulation
or verification in Python. Used this week for margin
distribution simulation (Day 1), token probability shift
demonstration (Day 3), and paired bootstrap null distribution
construction (Day 4). Every FDE should be able to write a
numpy simulation to verify a statistical claim before
publishing it.

---

**PEFT / HuggingFace**
https://huggingface.co/docs/peft

Why it matters: The standard library for LoRA adapter training
and inference. PeftModel.from_pretrained() loads adapters in
unmerged mode. merge_and_unload() fuses weights for production.
Understanding the difference between merged and unmerged
inference (numerically identical, performance different) is
essential before deploying any fine-tuned model.

---

**OpenRouter**
https://openrouter.ai

Why it matters: Unified API for accessing multiple LLM
providers with OpenAI-compatible interface. Supports tool
calling, streaming, and model routing. usage.completion_tokens_details.reasoning_tokens
field exposes reasoning token counts for thinking models —
essential for accurate cost accounting when using Qwen3,
o1, or DeepSeek-R1.

---

**Unsloth**
https://github.com/unslothai/unsloth

Why it matters: 2x faster LoRA fine-tuning with 70% less
memory on the same hardware. Makes Colab T4 fine-tuning
practical for models up to 7B parameters. Essential tool
for any FDE who needs to train a domain-specific adapter
without a GPU budget.

---

**GroupShuffleSplit (scikit-learn)**
https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.GroupShuffleSplit.html

Why it matters: Splits data while keeping groups intact —
essential for preventing augmentation-family memorization
from masquerading as generalization. Any FDE who trains on
augmented data and evaluates on a holdout split without
using GroupShuffleSplit risks inflated metrics.

---

## Engineering Patterns

---

**Position-Swap Audit**

The cheapest LLM judge bias diagnostic. Present each
candidate pair twice with order reversed. Measure flip
rate and slot_A_win_rate in both orderings.

Red flag: slot_A_win_rate > 0.65 in both orderings,
or flip_rate > 0.35.

Initial green flag: flip_rate < 0.2 and slot_A_win_rates
within 0.15 of each other.

Run this before reporting any result measured by a
comparative LLM judge.

---

**Margin Distribution Diagnostic**

Measure score_std of your deployment candidate pool
before retraining your critic.

score_std < 0.1 → GENERATION PROBLEM
Fix: diverse prompts, higher temperature, multiple models.
Do NOT retrain the critic.

score_std ≥ 0.1 → CRITIC PROBLEM
Fix: harder negatives, more training data.

Saves weeks of unnecessary retraining.

---

**Grouped Holdout Split**

When training on augmented data, all augmentations derived
from one original example must stay in the same split.

Use GroupShuffleSplit with group_key = original_example_id.

Sharp performance drop under grouped holdout vs standard
holdout indicates memorization not generalization.

---

**Gradient Norm by Module**

Log gradient norms during LoRA training, aggregated by
module group (attention: q/k/v/o vs MLP: gate/up/down).

Attention-side pressure → context-to-decision routing
MLP-side pressure → lexical/phrase transformation

High MLP pressure with weak grouped holdout performance
indicates surface pattern memorization.

---

**Paired Bootstrap for NLP Benchmarks**

The correct significance test for NLP evaluation measures
that are non-normal, bounded, or contain discrete jumps.

Use 10,000 bootstrap resamples. Report both p-value and
95% CI. Note sample size and its implications for power.

Rule of thumb: n=59 tasks detects medium effect sizes.
n=150-200 tasks needed for small effect sizes at alpha=0.05.

Never use a paired t-test on bounded evaluation scores
without first verifying normality.
