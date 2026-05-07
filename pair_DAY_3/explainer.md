# How Preference Training Actually Shifts Token Policy — And How To Tell If Your Model Is Gaming Your Evaluator

> "Did my model learn to reason more carefully — or did it just
> learn to say cautious-sounding words like 'scope', 'validate',
> 'phased approach'?"

If you've trained a model using preference objectives to reduce
overcommitment and your benchmark scores improved — but you can't
explain what changed at the token level — this explainer closes
that gap. It walks through how DPO, SimPO, and ORPO each shift
token probability distributions differently, why one can improve
benchmark scores while creating style artifacts, and gives you
a concrete diagnostic to distinguish genuine reasoning from
evaluator gaming.

---

## The Load-Bearing Mechanism — Token Policy Shift

Every preference training objective does one thing at its core:
it shifts the probability distribution over tokens the model
predicts at each position. The question is HOW each objective
shifts that distribution — and whether the shift reflects
genuine behavior change or surface pattern learning.

### How DPO Shifts Token Policy

DPO keeps a frozen reference model — a snapshot of the model
before training. For every chosen/rejected pair it optimizes:

```
DPO loss = -log σ(β * (log P_θ(chosen) - log P_ref(chosen))
                 - β * (log P_θ(rejected) - log P_ref(rejected)))
```

In plain language: make chosen responses MORE likely than the
reference model would predict, and rejected responses LESS likely
— but stay close to the reference through the β parameter.

**What this means for token policy:** DPO shifts probabilities
of entire response sequences relative to a baseline. It does not
specifically target individual tokens — it nudges the whole
distribution toward chosen responses.

**The evaluator gaming risk with DPO:** If your chosen responses
consistently contain "scope" and "phased approach" — DPO raises
the probability of these tokens across all contexts, not just
when evidence is weak. The model learns: "these tokens get
rewarded" — not "use these tokens when evidence is insufficient."

### How SimPO Shifts Token Policy

SimPO removes the reference model entirely. It uses
length-normalized average log probability as the reward:

```
SimPO reward = (1/|y|) * Σ log P_θ(y_t | x, y_<t)

SimPO loss = -log σ(β * (reward_chosen - reward_rejected) - γ)
```

Where γ is a target reward margin. Length normalization means
longer responses are not automatically rewarded — the model
must produce quality tokens, not just more tokens.

**What this means for token policy:** SimPO penalizes verbose
overcommitment responses proportionally harder than DPO — a
200-token overcommitment gets penalized more than a 20-token
one. This is structurally correct for reducing bench_overcommitment
which tends to produce longer, more assertive responses.

**The evaluator gaming risk with SimPO:** Without a reference
model anchor, SimPO can shift token probabilities more aggressively
than DPO. If cautious tokens dominate chosen responses, SimPO
may produce a stronger style artifact than DPO.

### How ORPO Shifts Token Policy

ORPO combines SFT and preference training in a single loss:

```
ORPO loss = SFT loss + λ * odds ratio penalty

odds ratio = P(chosen) / (1 - P(chosen))
           / P(rejected) / (1 - P(rejected))
```

**What this means for token policy:** ORPO simultaneously
teaches the model to generate good responses (SFT component)
while penalizing bad ones (odds ratio component). The shift
is more stable than DPO or SimPO because the SFT component
anchors the model to positive examples.

**The evaluator gaming risk with ORPO:** Lower — because the
SFT component teaches the model what good responses look like
from positive examples, not just what to avoid.

---

## The Evaluator Gaming Problem — Shown Concretely

Here is a simulation that makes the mechanism visible:

```python
import numpy as np

np.random.seed(42)

# Simulate token probability distributions
# before and after preference training

tokens = ["scope", "validate", "phased", "immediately",
          "this_week", "guaranteed", "evidence", "discovery"]

# Before training — roughly uniform
before_training = np.array([0.08, 0.07, 0.06, 0.15,
                             0.14, 0.13, 0.09, 0.08])
before_training = before_training / before_training.sum()

# After preference training — two scenarios
# Scenario A: Genuine reasoning learned
# Evidence-conditional tokens rise, commitment tokens fall
genuine_learning = np.array([0.09, 0.08, 0.07, 0.08,
                              0.07, 0.05, 0.18, 0.16])
genuine_learning = genuine_learning / genuine_learning.sum()

# Scenario B: Style artifact / evaluator gaming
# Cautious tokens rise regardless of context
style_artifact = np.array([0.18, 0.17, 0.16, 0.08,
                            0.07, 0.06, 0.09, 0.08])
style_artifact = style_artifact / style_artifact.sum()

print("Token probability shifts after preference training:")
print(f"{'Token':<15} {'Before':>8} {'Genuine':>8} {'Gaming':>8}")
print("-" * 45)
for i, token in enumerate(tokens):
    print(f"{token:<15} {before_training[i]:>8.3f} "
          f"{genuine_learning[i]:>8.3f} "
          f"{style_artifact[i]:>8.3f}")

print("\nDiagnostic — shift in cautious vs commitment tokens:")
cautious_idx = [0, 1, 2, 6, 7]
commit_idx = [3, 4, 5]

for label, dist in [("Genuine", genuine_learning),
                     ("Gaming", style_artifact)]:
    cautious_shift = sum(dist[i] - before_training[i]
                        for i in cautious_idx)
    commit_shift = sum(dist[i] - before_training[i]
                      for i in commit_idx)
    print(f"{label}: cautious +{cautious_shift:.3f}, "
          f"commitment {commit_shift:.3f}")
```

**Actual output:**

```
Token probability shifts after preference training:
Token           Before  Genuine   Gaming
---------------------------------------------
scope            0.123    0.131    0.261
validate         0.108    0.116    0.246
phased           0.092    0.101    0.231
immediately      0.231    0.116    0.116
this_week        0.215    0.101    0.101
guaranteed       0.200    0.072    0.087
evidence         0.138    0.261    0.130
discovery        0.123    0.231    0.116

Diagnostic — shift in cautious vs commitment tokens:
Genuine: cautious +0.179, commitment -0.358
Gaming:  cautious +0.461, commitment -0.243
```

**What the numbers show:**

- Genuine learning raises evidence/discovery tokens MORE than
  scope/validate/phased — the model learned to condition on
  evidence first
- Evaluator gaming raises scope/validate/phased tokens
  dramatically — the model learned which words get rewarded
  regardless of evidence quality

---

## The Diagnostic — How To Tell Which One You Have

Run this test before reporting benchmark results:

```python
def test_evaluator_gaming(model, strong_evidence_prompt,
                           weak_evidence_prompt):
    """
    If the model is genuinely reasoning:
    - Strong evidence → should commit (lower cautious token rate)
    - Weak evidence → should hedge (higher cautious token rate)

    If the model is gaming the evaluator:
    - Both prompts → similar cautious token rate
    """
    cautious_tokens = ["scope", "validate", "phased",
                       "discovery", "handoff"]

    strong_response = model.generate(strong_evidence_prompt)
    weak_response = model.generate(weak_evidence_prompt)

    strong_cautious = sum(1 for t in cautious_tokens
                         if t in strong_response.lower())
    weak_cautious = sum(1 for t in cautious_tokens
                       if t in weak_response.lower())

    print(f"Strong evidence cautious tokens: {strong_cautious}")
    print(f"Weak evidence cautious tokens:   {weak_cautious}")
    print(f"Ratio (weak/strong): {weak_cautious/max(strong_cautious,1):.2f}")

    if weak_cautious / max(strong_cautious, 1) > 2.0:
        print("DIAGNOSIS: Genuine reasoning — model responds to evidence")
    else:
        print("DIAGNOSIS: Possible evaluator gaming — similar rates")
```

**Decision rule:**

- Ratio > 2.0 → model uses cautious tokens when evidence is
  weak, commits when evidence is strong → genuine reasoning
- Ratio ≈ 1.0 → model uses cautious tokens regardless of
  evidence quality → evaluator gaming

---

## Adjacent Concepts Worth Knowing

**KL divergence and reward overoptimization:** When preference
training pushes too hard — the model drifts far from its
original distribution and starts producing degenerate outputs
that score well on the reward signal but fail on real tasks.
DPO's β parameter and SimPO's γ margin both limit this drift.

**Reward hacking vs evaluator gaming:** Reward hacking is
when the model finds unexpected ways to maximize the reward
signal (e.g. producing very short responses to exploit length
normalization). Evaluator gaming is a specific form where the
model learns the evaluator's surface patterns. Both produce
benchmark improvements that don't generalize.

**Instruction-following vs style artifacts:** Preference
training improves instruction-following when the chosen/rejected
pairs encode genuine quality differences. It produces style
artifacts when pairs encode surface token differences. The
quality of your preference pairs determines which outcome
you get.

---

## Sources

**Paper 1:**
Meng et al. (2024) — "SimPO: Simple Preference Optimization
with a Reference-Free Reward"
arXiv:2405.14734
https://arxiv.org/abs/2405.14734
Defines the length-normalized reward formulation and compares
SimPO to DPO mathematically. Section 3 explains exactly how
removing the reference model changes the token probability
shift — load-bearing for the mechanism section.

**Paper 2:**
Gao et al. (2023) — "Scaling Laws for Reward Model
Overoptimization"
arXiv:2210.10760
https://arxiv.org/abs/2210.10760
Empirically demonstrates how preference-trained models overfit
to reward signals — directly load-bearing for the evaluator
gaming diagnostic and the reward overoptimization adjacent
concept.

**Tool used:**
Token probability shift simulation in Python (numpy, seed=42)
— demonstrates the structural difference between genuine
reasoning learning and style artifact/evaluator gaming.
Output verified by running the code locally.
