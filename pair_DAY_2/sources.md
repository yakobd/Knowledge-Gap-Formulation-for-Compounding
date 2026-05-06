# Sources — Day 2

**Explainer written by:** Yakob Dereje
**For question asked by:** Yosef Zewdu

---

## Canonical Paper 1

**Schick et al. (2023) — "Toolformer: Language Models Can Teach
Themselves to Use Tools"**
arXiv:2302.04761
https://arxiv.org/abs/2302.04761

The foundational paper showing how models learn to generate
structured tool call tokens through training. Section 3 explains
the token-level mechanism — how API calls are represented as
text in training data and how models learn to reproduce them.
Load-bearing for the token generation mechanism section and
the explanation of why tool calling is token prediction not
decision making.

---

## Canonical Paper 2

**OpenAI (2023) — Function Calling Documentation**
Official API documentation
https://platform.openai.com/docs/guides/function-calling

Authoritative specification of how tool schemas are injected
into context, how the model generates tool_calls, and how
finish_reason distinguishes tool calls from final answers.
Load-bearing for the full token flow section and the hands-on
demonstration showing real tool_calls output.

---

## Tool / Pattern Used

**OpenAI-compatible client via OpenRouter — minimal tool
calling demonstration script**

Shows exactly what tool_calls contains after OpenRouter
intercepts the model's structured token stream, what
finish_reason looks like for tool calls vs final answers,
and how JSON arguments are parsed from the tool_calls field
that route_after_llm() reads at graph.py:783.

---

## Follow-On Reading

**Yao et al. (2023) — "ReAct: Synergizing Reasoning and Acting
in Language Models"**
arXiv:2210.03629
The next concept after tool calling — how models interleave
reasoning traces with tool calls to produce more reliable
multi-step agent behavior. Directly relevant to Yosef's
LangGraph multi-turn agent architecture.
