# What Your Model Actually Produces When It "Calls" a Tool

> "You can't reason about why the model sometimes ignores a tool
> without knowing what choosing a tool looks like structurally."

Your LangGraph agent routes on whether `tool_calls` exists in the
model's response. Your router at `agent/graph.py:783` branches on
this field every single turn. But what did the model actually
generate to create that field — and why does it sometimes generate
nothing at all?

This explainer works through the full mechanism: from tool schema
injection into the context window, to the structured tokens the
model generates, to how OpenRouter intercepts and parses them into
the `tool_calls` dict your router reads.

---

## The Load-Bearing Mechanism — Tool Calling Is Token Prediction

When you pass tool schemas to the model via OpenRouter, they do not
go into the model's weights or memory. They go into the **context
window** — the prompt the model reads before generating any tokens.

The context window for every turn in Yosef's agent looks like this:

```
┌─────────────────────────────────────────┐
│ SYSTEM PROMPT                           │
│ "You are a helpful sales agent..."      │
│                                         │
│ TOOLS AVAILABLE:                        │
│ - hubspot_upsert_contact                │
│   description: "add or update contact" │
│   parameters: {email, company...}       │
│                                         │
│ - send_email                            │
│   description: "send outreach email"   │
│   ...                                   │
│                                         │
│ USER MESSAGE:                           │
│ "Add John Smith to HubSpot"             │
└─────────────────────────────────────────┘
```

The model reads ALL of this — top to bottom — before generating
a single token. The tool schemas are just text. The model learned
during training that when tool schemas appear in context AND the
user request matches a tool description, the correct next tokens
to generate are a structured JSON object.

When the model "calls" hubspot_upsert_contact, it generates this:

```json
{
  "name": "hubspot_upsert_contact",
  "arguments": {
    "email": "john.smith@yellow.ai",
    "company": "Yellow.ai",
    "lifecycle": "lead"
  }
}
```

This is plain text. Not a magic token. Not a button press. Not a
decision. Just structured JSON text tokens that the model learned
to produce when the situation calls for a tool.

**The key architectural fact:** The model is not choosing a tool.
It is predicting the most probable next tokens given everything
in its context. Those tokens happen to be valid JSON that matches
a tool schema. OpenRouter intercepts this token stream, recognizes
the pattern, and converts it into the structured tool_calls object
your router reads.

---

## The Full Token Flow — Step by Step

Here is exactly what happens between your user message and your
router's branch decision:

```
Step 1: Tool schemas from tools.py injected into prompt
        → model reads them as plain text tokens

Step 2: User message added to context
        → "Add John Smith to HubSpot"

Step 3: Model generates structured JSON tokens
        → {"name": "hubspot_upsert_contact", "arguments": {...}}

Step 4: OpenRouter intercepts the token stream
        → parses the JSON structure
        → creates tool_calls object:
           response.choices[0].message.tool_calls = [
               {
                 "id": "call_abc123",
                 "type": "function",
                 "function": {
                   "name": "hubspot_upsert_contact",
                   "arguments": '{"email": "john.smith@yellow.ai"}'
                 }
               }
           ]

Step 5: route_after_llm() at graph.py:783 checks
        → if tool_calls → execute the tool
        → if no tool_calls → final answer

Step 6: Tool executes, result returned to model
        → model sees tool result, generates next response
```

The finish_reason field in the API response tells you which branch
was taken: finish_reason="tool_calls" means the model generated
a tool call. finish_reason="stop" means it gave a final answer.

---

## Hands-On Demonstration

This minimal script shows what tool_calls actually contains and
what finish_reason looks like when a model calls a tool:

```python
import os
import json
import openai

client = openai.OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key=os.getenv("OPENROUTER_API_KEY")
)

# Define a minimal tool schema
tools = [
    {
        "type": "function",
        "function": {
            "name": "hubspot_upsert_contact",
            "description": "Add or update a contact in HubSpot CRM",
            "parameters": {
                "type": "object",
                "properties": {
                    "email": {
                        "type": "string",
                        "description": "Contact email address"
                    },
                    "company": {
                        "type": "string",
                        "description": "Company name"
                    }
                },
                "required": ["email", "company"]
            }
        }
    }
]

# Send a request that should trigger the tool
response = client.chat.completions.create(
    model="openai/gpt-4o-mini",
    messages=[
        {"role": "user", "content": "Add john@yellow.ai from Yellow.ai to HubSpot"}
    ],
    tools=tools
)

message = response.choices[0].message
print(f"finish_reason: {response.choices[0].finish_reason}")
print(f"tool_calls present: {message.tool_calls is not None}")

if message.tool_calls:
    tool_call = message.tool_calls[0]
    print(f"tool name: {tool_call.function.name}")
    print(f"arguments: {tool_call.function.arguments}")
    args = json.loads(tool_call.function.arguments)
    print(f"parsed email: {args['email']}")
    print(f"parsed company: {args['company']}")
```

**Expected output:**

```
finish_reason: tool_calls
tool_calls present: True
tool name: hubspot_upsert_contact
arguments: {"email": "john@yellow.ai", "company": "Yellow.ai"}
parsed email: john@yellow.ai
parsed company: Yellow.ai
```

When finish_reason is "stop" instead of "tool_calls" — the model
gave a text answer and your router takes the final answer branch.

---

## Why The Model Sometimes Ignores a Tool

This is the production failure Yosef named — and now the mechanism
makes it explainable.

The model generates tool call tokens when two conditions are met:

1. A tool description clearly matches the user's request
2. The model's training strongly associates this situation with
   tool use rather than a text answer

When either condition fails — the model generates text tokens
instead of JSON tokens. Your router sees no tool_calls field and
takes the final answer branch — even when a tool should have run.

Three specific causes:

**Cause 1 — Vague tool description:**
If hubspot_upsert_contact is described as "manages contacts" —
the model cannot reliably match "add John Smith" to this tool.
Precise descriptions like "add or update a contact in HubSpot
CRM given an email and company name" produce reliable tool calls.

**Cause 2 — Ambiguous user request:**
"Handle the Yellow.ai situation" matches no tool description
clearly. The model defaults to a text answer because its training
associated vague requests with verbal responses not tool calls.

**Cause 3 — Temperature and sampling:**
At higher temperatures the model samples more randomly. A token
that begins a text answer ("I will...") can be sampled instead
of the opening brace of a JSON object ("{"). Once the first
token is sampled the model continues in that direction.

---

## Adjacent Concepts Worth Knowing

**Constrained decoding:** Some inference frameworks force the
model to generate valid JSON by restricting the token vocabulary
during generation. This eliminates sampling failures but requires
the framework to support it — OpenRouter does not do this by
default.

**Tool description engineering:** The most reliable way to
improve tool call rates is improving tool descriptions. A
description that names the exact trigger condition ("use this
when the user asks to add or update a CRM contact") outperforms
a generic description every time.

**Parallel tool calls:** Modern models can generate multiple
tool call JSON objects in a single response — calling
hubspot_upsert_contact and send_email in the same turn. Your
router needs to handle the case where tool_calls contains more
than one entry.

---

## Sources

**Paper 1:**
Schick et al. (2023) — "Toolformer: Language Models Can Teach
Themselves to Use Tools"
arXiv:2302.04761
https://arxiv.org/abs/2302.04761
The foundational paper showing how models learn to generate
structured tool call tokens through training. Section 3 explains
the token-level mechanism — how API calls are represented as
text in training data and how models learn to reproduce them.
Load-bearing for the token generation mechanism section.

**Paper 2:**
OpenAI (2023) — "Function Calling and Other API Updates"
Official documentation
https://platform.openai.com/docs/guides/function-calling
Authoritative specification of how tool schemas are injected
into context, how the model generates tool_calls, and how
finish_reason distinguishes tool calls from final answers.
Load-bearing for the full token flow section and the
hands-on demonstration.

**Tool used:**
OpenAI-compatible client via OpenRouter — minimal tool calling
script demonstrating real tool_calls output, finish_reason
values, and JSON argument parsing. Shows exactly what Yosef's
route_after_llm() router reads at line 783.
