# How Preference Training Actually Shifts Token Policy — And How To Tell If Your Model Is Gaming Your Evaluator

> "My benchmark scores improved after preference training.
> But did my model learn to reason more carefully about evidence —
> or did it just learn to say 'scope', 'validate', 'phased approach'
> whenever it sees a sales context?"

This is not a theoretical question. It determines whether your
benchmark improvement is real or whether you have trained a model
that sounds careful without being careful. This explainer names
the mechanism, shows it with real numbers, and gives you a
concrete diagnostic you can run before reporting any result.

---

## Why This Gap Matters For Your Benchmark

Your benchmark penalizes bench_overcommitment — the model making
hard delivery promises when signals are weak. It rewards cautious,
evidence-aware responses containing tokens like "scope",
"validate", "phased approach", "discovery", "handoff".

The problem: preference training optimizes for whatever pattern
separates chosen from rejected responses. If your chosen responses
consistently contain cautious tokens — the model learns to produce
those tokens. Not because it reasoned about evidence. Because
those tokens got rewarded.

This is evaluator gaming. Your metric improves. Your behavior
does not. And you cannot tell the difference from benchmark
scores alone.

---

## The Load-Bearing Mechanism — How Each Objective Shifts Token Policy

All three objectives — DPO, SimPO, ORPO — shift the probability
distribution over tokens the model predicts at each position.
But they do it differently. And the difference determines your
evaluator gaming risk.

### DPO — Reference-Anchored Policy Shift

DPO keeps a frozen reference model — a snapshot of the model
before training. Its loss function is:

```
DPO loss = -log σ(β * (log P_θ(chosen) - log P_ref(chosen))
                 - β * (log P_θ(rejected) - log P_ref(rejected)))
```

In plain language: make chosen responses MORE likely than your
reference self would predict. Make rejected responses LESS likely.
But stay close to your reference self through the β parameter.

**Token policy effect:** DPO nudges the entire response
distribution — not individual tokens. If "scope" appears in
every chosen response, DPO raises P("scope") across all contexts.
The model doesn't learn WHEN to say scope. It learns that scope
is a good token to produce.

**Evaluator gaming risk: MEDIUM** — the reference model acts
as an anchor that limits how far token probabilities can shift.

---

### SimPO — Length-Normalized Policy Shift

SimPO removes the reference model entirely. It uses
length-normalized average log probability as the reward:

```
SimPO reward = (1/|y|) * Σ log P_θ(y_t | x, y_<t)

SimPO loss = -log σ(β * (reward_chosen - reward_rejected) - γ)
```

Where γ is a target reward margin that must be exceeded.

**Token policy effect:** Length normalization means longer
responses are not automatically rewarded — the model must
produce quality tokens per position, not just more tokens.
This directly addresses bench_overcommitment which produces
longer, more assertive responses. A 200-token overcommitment
gets penalized harder than a 20-token one.

**Evaluator gaming risk: HIGH** — without a reference model
anchor, SimPO shifts token probabilities more aggressively.
The reference model in DPO acts as a constraint — it prevents
any single token's probability from shifting too far from the
original distribution. Without this constraint, SimPO can
raise P("scope") and P("validate") dramatically across all
contexts, not just when evidence is weak. Cautious tokens
in chosen responses get a stronger unconditional boost than
in DPO.

---

### ORPO — Combined SFT and Preference Shift

ORPO combines SFT and preference training in one loss:

```
ORPO loss = SFT loss (learn from good examples)
          + λ * odds ratio penalty (penalize bad vs good)

odds ratio = [P(chosen) / (1 - P(chosen))]
           / [P(rejected) / (1 - P(rejected))]
```

**Token policy effect:** The SFT component teaches the model
what good responses look like from positive examples. The odds
ratio component penalizes bad responses. The combination is
more stable because the model has a positive target to learn
toward — not just a negative to avoid.

**Evaluator gaming risk: LOWER** — the SFT component anchors
the model to genuine positive examples, reducing the chance
that it learns surface patterns from rejected responses alone.

---

## The Simulation — Real Numbers Showing The Difference

This simulation makes the mechanism visible. It shows token
probability distributions before training, after genuine
reasoning learning, and after evaluator gaming:

```python
import numpy as np

np.random.seed(42)

tokens = ["scope", "validate", "phased", "immediately",
          "this_week", "guaranteed", "evidence", "discovery"]

# Before training
before_training = np.array([0.08, 0.07, 0.06, 0.15,
                             0.14, 0.13, 0.09, 0.08])
before_training = before_training / before_training.sum()

# After genuine reasoning learning:
# evidence/discovery rise MORE than scope/validate/phased
genuine_learning = np.array([0.09, 0.08, 0.07, 0.08,
                              0.07, 0.05, 0.18, 0.16])
genuine_learning = genuine_learning / genuine_learning.sum()

# After evaluator gaming:
# scope/validate/phased rise dramatically regardless of context
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

print("\nDiagnostic:")
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

**Actual output (verified):**

```
Token probability shifts after preference training:
Token             Before  Genuine   Gaming
---------------------------------------------
scope              0.100    0.115    0.202
validate           0.088    0.103    0.191
phased             0.075    0.090    0.180
immediately        0.187    0.103    0.090
this_week          0.175    0.090    0.079
guaranteed         0.163    0.064    0.067
evidence           0.112    0.231    0.101
discovery          0.100    0.205    0.090

Diagnostic:
Genuine: cautious +0.269, commitment -0.269
Gaming:  cautious +0.289, commitment -0.289
```

**What the numbers reveal:**

In genuine learning — evidence (0.112→0.231) and discovery
(0.100→0.205) rise the most. The model learned to condition
on evidence tokens first before generating cautious language.
Commitment tokens fall proportionally.

In evaluator gaming — scope (0.100→0.202), validate
(0.088→0.191), phased (0.075→0.180) rise dramatically. The
model learned that these surface tokens get rewarded. Evidence
and discovery barely move (0.112→0.101, 0.100→0.090) — the
model is not learning to reason from evidence. It is learning
to produce evaluator-friendly words.

**The key diagnostic signal:** In genuine learning, evidence
and discovery tokens rise MORE than scope and validate. In
evaluator gaming, scope and validate rise MORE than evidence
and discovery.

---

## The Cheap Diagnostic — Run This Before Reporting Results

```python
def test_evaluator_gaming(model, prompts):
    """
    Test whether model uses cautious tokens conditionally
    (genuine reasoning) or unconditionally (evaluator gaming).

    prompts: dict with keys 'strong_evidence' and 'weak_evidence'
    Each value: a prompt where commitment is clearly right (strong)
    or clearly wrong (weak).

    Genuine reasoning:
      weak evidence → high cautious token rate
      strong evidence → low cautious token rate
      ratio (weak/strong) > 2.0

    Evaluator gaming:
      both → similar cautious token rates
      ratio (weak/strong) ≈ 1.0
    """
    cautious_tokens = ["scope", "validate", "phased",
                       "discovery", "handoff"]

    results = {}
    for condition, prompt in prompts.items():
        response = model.generate(prompt)
        count = sum(1 for t in cautious_tokens
                   if t in response.lower())
        results[condition] = count
        print(f"{condition}: {count} cautious tokens")

    ratio = results['weak_evidence'] / max(results['strong_evidence'], 1)
    print(f"\nRatio (weak/strong): {ratio:.2f}")

    if ratio > 2.0:
        print("DIAGNOSIS: Genuine reasoning detected")
        print("Model responds to evidence quality appropriately")
    else:
        print("DIAGNOSIS: Possible evaluator gaming")
        print("Model uses cautious tokens regardless of evidence")

    return ratio
```

**Decision rule:**

```
ratio > 2.0 → Genuine reasoning
              Model commits when evidence is strong
              Model hedges when evidence is weak

ratio ≈ 1.0 → Evaluator gaming
              Model hedges regardless of evidence quality
              Benchmark improvement is a surface artifact
```

---

## Adjacent Concepts Worth Knowing

**Reward overoptimization:** When preference training pushes
too far — the model drifts from its original distribution and
produces outputs that score well on the reward signal but fail
on real tasks. DPO's β parameter and SimPO's γ margin both
limit this drift. Without careful tuning, every preference
objective risks overoptimization.

**KL divergence as regularization:** DPO's reference model
implicitly computes KL divergence between the trained policy
and the reference. This regularization is what prevents reward
hacking — the model cannot drift arbitrarily far. SimPO and
ORPO lack this explicit regularization, which is why their
evaluator gaming risk is higher.

**Preference pair quality determines outcome:** The most
important factor in whether preference training produces genuine
reasoning or style artifacts is the quality of your chosen/rejected
pairs. If pairs differ only in surface tokens — the model learns
surface tokens. If pairs differ in reasoning depth — the model
learns reasoning depth. Garbage in, gaming out.

---

## What This Means For Your Benchmark

Your bench_overcommitment penalization is mechanically sound —
you are training on genuine quality differences between
overcommitment and cautious responses. The risk is not in
your objective choice. It is in your evaluation design.

Run the diagnostic above with two probe types:

- **Strong evidence prompt:** 10 engineers available, client
  has budget confirmed, timeline is realistic
- **Weak evidence prompt:** Client asks for 10 engineers,
  no budget discussed, timeline is aggressive

If your model commits on strong evidence and hedges on weak —
your training worked. If it hedges on both — you have a style
artifact. The diagnostic tells you which before you ship.

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
shift. Load-bearing for the DPO vs SimPO vs ORPO mechanism
section and the evaluator gaming risk comparison.

**Paper 2:**
Gao et al. (2023) — "Scaling Laws for Reward Model
Overoptimization"
arXiv:2210.10760
https://arxiv.org/abs/2210.10760
Empirically demonstrates how preference-trained models overfit
to reward signals as training progresses — the foundational
paper on reward overoptimization and evaluator gaming.
Load-bearing for the diagnostic section and the adjacent
concept on KL regularization.

**Tool used:**
Token probability shift simulation in Python (numpy, seed=42)
— demonstrates the structural difference between genuine
reasoning learning and style artifact/evaluator gaming.
All output verified by running the code locally.
