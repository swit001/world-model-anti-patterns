# Bad Example: Transition-less Action

This is what it looks like when actions are exposed as raw tool functions with no declared source state, target state, or guards. The agent can fire any function at any time.

---

## Tool definition

```python
tools = [
    {
        "name": "refundPayment",
        "description": "Refunds the payment for an order.",
        "parameters": {
            "order_id": {"type": "string"}
        }
    },
    {
        "name": "cancelOrder",
        "description": "Cancels an order.",
        "parameters": {
            "order_id": {"type": "string"}
        }
    }
]
```

## Agent invocation

```
User: The customer for order #8821 wants a refund.

Agent: I'll process the refund now.

[calls refundPayment(order_id="ord-8821")]
```

---

## What's wrong here

There is no declared:

- **Source state** — `refundPayment` can be called regardless of whether the order is `PENDING`, `SHIPPED`, `DELIVERED`, or already `REFUNDED`
- **Target state** — nothing declares what state the entity moves to after this call
- **Guards** — no conditions are checked before the function fires

This leads to:
- Refunding an order that was already refunded
- Refunding an order that has not been delivered yet
- Canceling an order that has already shipped
- An audit log of function calls with no record of state transitions

---

## The key failure

```python
refundPayment(order_id="ord-8821")
```

No `from` state. No `to` state. No guards. The function is a side effect with no relationship to entity state.

---

See [anti-patterns/02-transition-less-action.md](../../anti-patterns/02-transition-less-action.md) for the full analysis and fix.
