# Anti-Pattern 02: Transition-less Action

## Symptom

Action functions like `refundPayment()`, `cancelOrder()`, or `escalateTicket()` are exposed to the agent as tools. No declaration of which states are valid entry points, no declaration of the resulting state.

## Bad example

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

The agent can call either function at any time, from any state, for any reason.

## Why it fails

Without declared transitions, the agent has no idea:
- Whether the action is valid given the current state
- What state the entity will be in after the action
- Whether a prior action already moved the entity to a terminal state

This leads to:
- Refunding an order that was already refunded
- Canceling an order that has already shipped
- Chaining actions in an order that produces an inconsistent state
- No audit trail of state changes — just a log of function calls

The action is a side effect with no declared relationship to entity state. The system has no way to enforce correctness, and the agent has no way to reason about it accurately.

## World-model fix

Define every action as a transition with an explicit `from` state, `to` state, and the guards that must pass. The agent should only be offered transitions that are valid for the current entity state.

This is the T in ESTC: **Transition**.

A transition is not just a function. It is a declaration of:
- What state you must be in to trigger it
- What state you will be in after it succeeds
- What must be true before it fires (guards)

## ESTC lens

**Entity:** `Order`  
**State:** `DELIVERED`, `REFUND_REQUESTED`, `REFUNDED`, `CANCELLED`  
**Transition:** `RequestRefund` — from: `DELIVERED`, to: `REFUND_REQUESTED`  
**Constraint:** Guards evaluated before transition fires  

## Good example

```yaml
transitions:
  - name: RequestRefund
    from: DELIVERED
    to: REFUND_REQUESTED
    guards:
      - refund_window_open
      - item_is_refundable

  - name: ApproveRefund
    from: REFUND_REQUESTED
    to: REFUNDED
    guards:
      - refund_amount_valid

  - name: CancelOrder
    from: [PENDING, PROCESSING]
    to: CANCELLED
    guards:
      - payment_not_captured
```

The agent is only offered `RequestRefund` if the order is currently in state `DELIVERED`. It cannot call `ApproveRefund` from `DELIVERED` — the transition doesn't exist.

## Takeaway

An action without a declared transition is a footgun with no safety.

## Examples

- Bad: [examples/bad/transition-less-action.md](../examples/bad/transition-less-action.md)
- Good: [examples/good/transition-defined-action.yaml](../examples/good/transition-defined-action.yaml)
