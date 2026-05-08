# Evening Call Summary — Day 4 (Evaluation and Statistics)

**Date:** May 8, 2026
**Triad:** Yakob Dereje, Nebiyou Abebe, Rafia Kedir
**Prepared by:** Yakob Dereje | **Confirmed by:** Nebiyou & Rafia

Nebiyou's explainer on LLM-as-a-judge biases covered all
three mechanisms at the token level and ran two real
experiments — a live position-swap audit (n=5, flip_rate=0.20,
slot_A_win_rate jumped 0.20 to 0.60) and a length-bias screen
on Week 11 artifacts showing negative length-score correlation
— giving Yakob a concrete minimum check package to run before
treating Delta A as a defensible model-quality gain.

Yakob's explainer on paired bootstrap null distribution
construction was revised after Rafia's feedback to make the
60/40 blending asymmetry more explicit — the core problem is
not variance reduction alone but that baseline and trained
conditions use different variance regimes, making the bootstrap
comparison structurally unfair.

The most important insight from the evening call was that both
questions share the same underlying problem: evaluation coupling
— the measurement system shares structure with the thing being
measured. Nebiyou's judge may favor outputs matching its
preferred slot or style; Rafia's blended scorer creates
asymmetric variance across conditions.

Yakob confirmed gap closed ✅ — position bias, length bias,
and self-preference bias are now understood mechanically, and
the minimum defensible audit package is concrete and runnable.

Rafia confirmed gap closed ✅ — paired bootstrap null
distribution construction is now mechanically understood and
the 60/40 blending asymmetry is identified as a concrete fix.
