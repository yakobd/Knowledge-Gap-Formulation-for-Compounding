# Evening Call Summary — Day 2 (Agent and Tool-Use Internals)

**Date:** May 6, 2026
**Pair:** Yakob Dereje & Yosef Zewdu
**Prepared by:** Yosef Zewdu | **Confirmed by:** Yakob Dereje

Yakob's explainer on tool-calling token mechanics was revised
to make the translation chain concrete — the original draft
described the process abstractly and feedback pushed for naming
exactly what token sequence the model emits and where OpenRouter
intercepts it before it reaches route_after_llm().

Yosef's explainer on reasoning tokens was revised twice — first
to replace the think tag heuristic with a direct read of
usage.completion_tokens_details.reasoning_tokens, then to add
the actual audit results (96.9%, 32:1 ratio) from
held_out_traces.jsonl replacing estimated figures.

Yosef confirmed Yakob's explainer closed his gap — the
upstream/downstream distinction in tool calling is now clear.

Yakob confirmed Yosef's explainer closed his gap — the
difference between additional_cost_usd: 0.0 and the real cost
of running a deliberative prompt in a thinking model is now
fully understood and precisely quantifiable.

Both gaps fully closed ✅
