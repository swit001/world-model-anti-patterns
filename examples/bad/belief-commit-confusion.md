# Bad Example: Belief-Commit Confusion

This is what it looks like when an agent treats its own inference as committed fact. The agent estimates the entity state from user input and acts on that estimate without reading the system of record.

---

## Prompt

```
User: The customer says the package arrived. Should we issue the refund?

Agent: The order looks delivered, so I can refund it.

[calls refundPayment(order_id="ord-8821")]
```

---

## What's actually in the system of record

```
order.status = SHIPPED
```

The order has not been marked `DELIVERED`. The carrier has not confirmed delivery. The agent inferred a delivery state from the customer's message and acted on it as if it were a committed fact.

---

## What's wrong here

The agent conflated two fundamentally different things:

| | Type | Source | Reliability |
|---|---|---|---|
| "The order looks delivered" | Believed state (NWM) | Customer message + LLM inference | Can be wrong |
| `order.status = SHIPPED` | Committed state (SWM) | System of record | Authoritative |

The agent never queried committed state. It used its own belief as the source of truth. That belief was wrong.

---

## The key failure

```
"The order looks delivered, so I can refund it."
```

"Looks delivered" is not `status: DELIVERED` written to the database by a confirmed delivery event. One is an estimate. The other is a fact. Acting on the estimate as if it were the fact is the anti-pattern.

---

See [anti-patterns/04-belief-commit-confusion.md](../../anti-patterns/04-belief-commit-confusion.md) for the full analysis and fix.
