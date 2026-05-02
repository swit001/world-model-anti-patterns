# Anti-Pattern 03: Guard as Prompt

## Symptom

Business rules and safety constraints are written as prompt instructions. The agent is told what it should or should not do in natural language, and expected to enforce this consistently at runtime.

## Bad example

```
System: You are a refund agent. Follow these rules:
  - Never process a refund if the order is more than 7 days old.
  - Do not refund non-refundable items.
  - If the refund amount exceeds $500, escalate to a human.
  - Only process refunds for orders in delivered status.

User: Process a refund for order #8821.
```

These rules live in the prompt. The agent interprets them. The agent enforces them. The agent can forget them, misapply them, or be talked out of them.

## Why it fails

LLMs do not reliably enforce hard rules. They are probabilistic systems. A sufficiently compelling user message, an unusual edge case, or a long conversation that pushes the rules out of the context window can all lead to a rule being bypassed.

Specific failure modes:
- A persuasive user says "this is an exception" and the model agrees
- The model miscalculates the date difference and decides 8 days is "close enough"
- The rule appears late in a long system prompt and gets underweighted
- Two rules appear to conflict and the model resolves it in the wrong direction

Prompt-based guards are soft. Business rules are hard. The gap between them is where production incidents happen.

## World-model fix

Guards must be executable conditions, not prose. Implement them in code, schema validation, or a DSL that runs before any transition fires. The agent does not evaluate the guard — it receives a pass or fail result.

This is the C in ESTC: **Constraint**.

A constraint is not a suggestion. It is a predicate that must evaluate to `true` before a transition is permitted. If it evaluates to `false`, the transition is blocked — regardless of what the agent thinks.

## ESTC lens

**Entity:** `Order`  
**State:** `DELIVERED`  
**Transition:** `RequestRefund`  
**Constraint:** `refund_window_open`, `item_is_refundable`, `amount_within_auto_approve_limit` — evaluated in code, not by the LLM  

## Good example

```yaml
guards:
  refund_window_open:
    type: temporal
    rule: "order.delivered_at + 7d >= now()"
    error: "Refund window expired"

  item_is_refundable:
    type: attribute
    rule: "order.item.refundable == true"
    error: "Item is marked non-refundable"

  amount_within_auto_approve_limit:
    type: numeric
    rule: "order.total <= 500"
    on_fail: escalate
    error: "Amount exceeds auto-approve threshold"
```

Each guard is a named, executable predicate with a clear failure message. The LLM never evaluates these — the runtime does. The LLM receives the result.

## Takeaway

A constraint written in a prompt is a preference. A constraint written in code is a rule.
