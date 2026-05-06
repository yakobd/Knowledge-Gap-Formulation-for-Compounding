# Grounding Commit — Day 2

**Asker:** Yakob Dereje
**Repository edited:** yakobd/The-Conversion-Engine
**File edited:** agent/act4_mechanism.py
**Section:** get_mechanism_metadata() return value

## Before (original text)

```python
return {
    "mechanism_name": "confidence_aware_abstention_v1",
    "mechanism_version": "1.0",
    "target_failure": "signal_over_claiming_and_dual_control",
    "probe_categories": ["signal_over_claiming", "dual_control_coordination"],
    "mechanism_type": "prompt_injection",
    "additional_llm_calls": 0,
    "additional_cost_usd": 0.0,
    "description": (
        "Injects confidence-aware behavioral rules into agent system prompt. "
        "Forces evidence check before claims, abstention on low confidence, "
        "and grounded language. Zero additional LLM calls."
    )
}
```

## After (replacement text)

```python
return {
    "mechanism_name": "confidence_aware_abstention_v1",
    "mechanism_version": "1.0",
    "target_failure": "signal_over_claiming_and_dual_control",
    "probe_categories": ["signal_over_claiming", "dual_control_coordination"],
    "mechanism_type": "prompt_injection",
    "additional_llm_calls": 0,
    "additional_cost_usd": 0.0,
    "reasoning_token_note": (
        "additional_cost_usd measures extra LLM calls only — correctly zero. "
        "When run on a thinking model (qwen3-next-80b-a3b-thinking), this "
        "mechanism's deliberative prompt causes reasoning tokens to constitute "
        "~97% of output tokens (32:1 reasoning:answer ratio measured across "
        "30 held-out simulations). Reasoning tokens are billed at output token "
        "rates. Total mechanism cost = base model cost × ~32x output multiplier. "
        "Use usage.completion_tokens_details.reasoning_tokens to audit."
    ),
    "description": (
        "Injects confidence-aware behavioral rules into agent system prompt. "
        "Forces evidence check before claims, abstention on low confidence, "
        "and grounded language. Zero additional LLM calls. Note: deliberative "
        "prompts in thinking models generate significant reasoning token volume "
        "— see reasoning_token_note for cost implications."
    )
}
```

## Why This Changed

Before Day 2 I believed additional_cost_usd: 0.0 fully captured
the mechanism's cost. I had no knowledge of reasoning trace tokens
or that thinking models bill them at output token rates.

After reading Yosef's explainer I understand that my confidence-
aware prefix — by asking the model to evaluate evidence, state
confidence, and consider abstention before every action — caused
the model to generate extensive internal reasoning. Across 30
held-out simulations: 377,285 reasoning tokens vs 11,902 answer
tokens. A 32:1 ratio. The original metadata was structurally
correct but materially incomplete for anyone trying to budget
the mechanism's real cost.

The new reasoning_token_note field gives any engineer reading
the metadata the information they need to correctly estimate
cost when running this mechanism on a thinking model.
