# Tweet Thread — Day 2: Agent and Tool-Use Internals

---

**Tweet 1:**
Your LangGraph agent routes on whether tool_calls exists.
But what did the model actually generate to create that field?

It's not a magic token.
It's not a real decision.

Here's what actually happens at the token level. 🧵

---

**Tweet 2:**
When you pass tool schemas to the model — they don't go into
its weights or memory.

They go into the PROMPT.
The model reads them as plain text on every single turn.

Tool calling starts as a reading comprehension problem,
not a decision problem.

---

**Tweet 3:**
When the model "calls" hubspot_upsert_contact — it generates
plain JSON text tokens:

{"name": "hubspot_upsert_contact",
"arguments": {"email": "john@yellow.ai", "company": "Yellow.ai"}}

That's it. No magic token. No button press.
Just structured text the model learned to produce.

OpenRouter intercepts this, parses it, creates tool_calls.
Your router reads tool_calls. Tool executes.

---

**Tweet 4:**
Why does the model sometimes ignore a tool entirely?

Three reasons:

1. Vague tool description → model can't match request to tool
2. Ambiguous user request → model defaults to text answer
3. High temperature → model samples "I will..." instead of "{"
   — first token determines the path

finish_reason="tool_calls" → tool ran
finish_reason="stop" → model gave text answer instead

---

**Tweet 5:**
The fix is not in your router. It's in your tool descriptions.

"manages contacts" → unreliable
"add or update a contact in HubSpot given email and company"
→ reliable

The model matches requests to tools through description quality
— not through logic or reasoning.

Tool calling is token prediction.
Description quality is your reliability lever.

---

**Tweet 6:**
Full explainer with token flow diagram, working code showing
real tool_calls output, and the three causes of tool-ignore
failures:

https://yakobdereje.substack.com/p/what-your-model-actually-produces

Sources:
- Schick et al. 2023 (Toolformer) — arxiv.org/abs/2302.04761
- OpenAI Function Calling docs — platform.openai.com/docs/guides/function-calling
