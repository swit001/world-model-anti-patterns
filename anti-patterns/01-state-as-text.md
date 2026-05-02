# Anti-Pattern 01: State as Text

## Symptom

Developers pass order summaries, ticket descriptions, or conversation history into a prompt and ask the agent to decide what state the entity is in. The agent's conclusion becomes the working state.

## Bad example

```
System: You are a refund agent.
User: The customer says they received the order last week and want a refund.
      Here is the order history: "Shipped on April 20. Driver confirmed drop-off."

Agent: The order seems delivered, so I'll go ahead and process the refund.
```

The agent read natural language and inferred a state. It never read the committed `order.status` field from the system of record.

## Why it fails

Natural language descriptions are ambiguous, stale, and unverifiable. "Seems delivered" is not the same as `status: DELIVERED` written to the database by a confirmed delivery event. The agent is acting on a belief, not a fact.

Common failures:
- Order is still in transit but the driver note is misleading
- Order was already refunded — the history doesn't mention it
- The customer's description of events is inaccurate

The agent has no way to detect any of these because it never checked the source of truth.

## World-model fix

State must be read from the committed system of record before any action is taken. The agent may summarize or reason about natural language, but it must resolve entity state from a structured, authoritative source — not infer it from prose.

Separate two things clearly:
- **Narrative world model (NWM):** what the LLM understands from conversation and context
- **Structural world model (SWM):** what the system of record actually says

The SWM always wins when they conflict.

## ESTC lens

**Entity:** `Order`  
**State:** Must be read from `order.status` in the system of record — not inferred from conversation  
**Transition:** `RequestRefund` is only valid from committed state `DELIVERED`  
**Constraint:** Agent cannot act until committed state is verified  

## Good example

```yaml
entity: Order
id: "ord-8821"
committed_state:
  status: DELIVERED
  delivered_at: "2026-04-20T14:32:00Z"
  refund_status: null

action: RequestRefund
pre_check:
  source_state: DELIVERED
  refund_status: null
result: ALLOW
```

The agent reads `committed_state.status`, confirms it is `DELIVERED`, and confirms no refund is already in progress before proceeding.

## Takeaway

An agent that infers state from text is not reading the world — it is imagining it.

## Examples

- Bad: [examples/bad/state-as-text.md](../examples/bad/state-as-text.md)
- Good: [examples/good/estc-refund-world.yaml](../examples/good/estc-refund-world.yaml)
