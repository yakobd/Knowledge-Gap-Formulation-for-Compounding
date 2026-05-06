# Morning Call Summary — Day 2 (Agent and Tool-Use Internals)

**Date:** May 6, 2026
**Pair:** Yakob Dereje & Yosef Zewdu
**Duration:** ~20 minutes
**Prepared by:** Yakob Dereje | **Confirmed by:** Yosef Zewdu

## What Was Ambiguous in the Original Drafts

**Yakob's original draft question (before the call):**
> "In my Week 10 mechanism run I used qwen3-next-80b-a3b-thinking
> and recorded additional_cost_usd: 0.0 in act4_mechanism.py.
> What are reasoning trace tokens and are they billed separately?"

Too narrow — it only asked about billing without connecting to
the deeper agent and tool-use internals topic. It also didn't
name the specific cost figure at stake or explain why the gap
mattered beyond a single number.

**Yosef's challenge during the call:**
Yosef asked: "Are you asking about how thinking models generate
reasoning tokens mechanically, or just about whether you were
charged for them? Because those are two different questions and
one is much more useful than the other for an FDE."

**Yakob's rewrite after the challenge:**
Rewritten to connect the reasoning token generation mechanism
to the agent and tool-use internals topic — specifically how
thinking tokens interact with tool-use decisions inside a
reasoning model. Named the exact artifact (act4_mechanism.py
additional_cost_usd: 0.0), the exact figure at stake ($6.17
total LLM spend), and the exact consequence: the cost table
in memo.pdf cannot be defended without knowing the reasoning
token vs output token breakdown.

---

**Yosef's original draft question (before the call):**
> "How does function calling work at the token level in my
> LangGraph agent when it routes to hubspot_upsert_contact?"

Broad enough to go in multiple directions — the token
generation, the OpenRouter parsing, the LangGraph routing
logic, or the tool schema design. No specific artifact pointer
named. No specific failure mode connected.

**Yakob's challenge during the call:**
Yakob noted a related confusion from his own Week 10 system:
"In my e2e_thread.py, my agent calls HubSpot, Cal.com, and
Resend — but Python makes every tool call directly. The model
never issues a tool call. I described my system as an agent
but it has no LLM function calling at all. So I was also
confused — is the model actually deciding to call a tool, or
is it just predicting structured tokens that look like a
tool call?"

This challenge sharpened Yosef's question significantly —
forcing it to name the exact mechanism (token generation vs
decision making) and connect it to a specific artifact
(route_after_llm() at agent/graph.py:783 and the 9 schemas
in agent/tools.py:15–203).

**Yosef's rewrite after the challenge:**
Named the exact router function (route_after_llm() line 783),
the exact tool (hubspot_upsert_contact), the exact artifact
files (graph.py and tools.py), and the exact consequence:
"you can't reason about why the model sometimes ignores a
tool without knowing what choosing a tool looks like
structurally."

## Outcome

Both questions finalized by end of call. Each question is
unambiguous to the other partner. Questions committed final.
