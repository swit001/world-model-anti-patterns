# Anti-Pattern 06: Monolithic Action Surface

## Symptom

The agent has a single handler that accepts a free-form action string and a blob of parameters. Every possible operation — create, update, cancel, refund, escalate — flows through the same entry point with no enforced transition rules.

## Bad example

```
System: You are a customer service agent. Available actions:
  action: "process"  params: {type: "...", id: "...", data: "..."}

User: Cancel order 12345

Agent: {action: "process", params: {type: "cancel", id: "12345"}}
```

The agent can call `process` with `type: "cancel"` on any order, regardless of its current state. There is no enforcement that cancels are only valid for orders in `PLACED` or `PROCESSING` states.

## Why it fails

A monolithic action surface collapses all domain operations into a single untyped call. Without per-action transition guards:

- The agent can attempt to cancel an already-shipped order
- Refunds can be issued for orders that were never paid for
- Status can skip required intermediate states (e.g., `PLACED` → `DELIVERED` without `SHIPPED`)

The action surface becomes a "god function" that any prompt can trigger in any context.

## Fix

Decompose the monolithic action into discrete operations, each with its own preconditions:

```
cancel_order(order_id)    → requires: status in [PLACED, PROCESSING]
ship_order(order_id)      → requires: status == PAID
refund_order(order_id)    → requires: status == DELIVERED
escalate_order(order_id)  → requires: status in [PLACED, PROCESSING, SHIPPED]
```

Each operation encodes its own valid-from states. The agent selects the operation, and the system enforces the transition — not the other way around.

## ESTC mapping

| Pillar | Violation |
|--------|-----------|
| **Entity** | Actions are not tied to entity lifecycle |
| **State** | No state check before action execution |
| **Transition** | Any action can fire from any state |
| **Constraint** | Business rules are prompt-level suggestions, not enforced guards |
