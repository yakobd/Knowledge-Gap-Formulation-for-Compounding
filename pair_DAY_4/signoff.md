# Sign-off — Day 4

**Asker:** Yakob Dereje
**Explainer written by:** Nebiyou Abebe
**Gap verdict:** CLOSED ✅

## What I Now Understand That I Did Not Before

**Before today:**
I reported Delta A: +0.263 (p<0.0001) and inter-rater
agreement ≥80% — but I could not explain whether my judge
had position bias, length bias, or self-preference bias.
I knew the 80% agreement number but could not connect it
to whether my Delta A result was a genuine quality gain
or a judge artifact.

**After today, I understand four things precisely:**

**1. 80% inter-rater agreement does not prove bias absence.**
Agreement answers whether the judge is consistent with
the reference labeling protocol. It does not answer whether
the judge is invariant to presentation order, output length,
or model-family familiarity. These are different validity
checks — both are necessary.

**2. Each bias manifests at the token level during inference.**
Position bias: slot identity becomes a feature in the judge's
attention — earlier text and candidate labels shift logit
probabilities before the model grounds the decision in rubric
evidence. Length bias: longer outputs contain more tokens
resembling quality cues — connective phrases, caveats,
rubric keywords — making the decoder more likely to emit
high-score tokens. Self-preference: familiar text has lower
perplexity for the judge, activating quality-associated
continuations regardless of authorship.

**3. Cross-model judging partially mitigates self-preference
but does not eliminate it.**
Using different model families prevents direct self-grading
but does not address shared post-training style or familiarity
bias. Wataoka et al. show judges can favor lower-perplexity
text regardless of whether they literally generated it.
My judge rotation is directionally right — not a full proof
of neutrality.

**4. My Delta A is not invalid — it is under-audited.**
The defensible claim is: "Delta A = +0.263 is a strong
result under the current judge. Because the judge was not
audited for position, length, or self-preference bias,
systematic judge bias could explain part of the lift."
The minimum fix: position swap on 20+ pairs, length-score
correlation check, lineage separation confirmation.

## What Changed in My Existing Work

Updated model_card.md in yakobd/The_Sales_Agent_Evaluation_Bench
to replace the bare inter-rater agreement claim with a
technically grounded bias audit section. See
grounding_commit.md for full details.
