# Morning Call Summary — Day 2 (Agent and Tool-Use Internals)

**Date:** May 6, 2026
**Pair:** Yakob Dereje & Yosef Zewdu
**Duration:** ~20 minutes
**Prepared by:** Yosef Zewdu | **Confirmed by:** Yakob Dereje

Yakob's original question asked whether reasoning trace tokens
are billed separately — Yosef challenged this as a pure billing
question and pushed for the mechanistic version: what does a
deliberative prompt actually do to a thinking model's output
budget, grounded in act4_mechanism.py's additional_cost_usd: 0.0
and the $6.17 total spend in invoice_summary.json.

Yosef's original question asked broadly how function calling
works at the token level — Yakob challenged this by raising his
own confusion from e2e_thread.py, where Python makes every tool
call directly and the model never issues a tool call at all,
forcing the question to name the exact structural distinction
between token-level tool calling and Python orchestration.

Both questions were sharpened to name specific artifacts:
act4_mechanism.py and held_out_traces.jsonl for Yakob,
route_after_llm() at graph.py:783 and tools.py lines 15-203
for Yosef.

We agreed the bar for each explainer was a concrete runnable
demonstration — an audit of held_out_traces.jsonl for Yakob's
question and real tool_calls output for Yosef's question.

Both questions committed final by end of call.