# Anti-Pattern 04: Belief-Commit Confusion

## Symptom

The agent reasons about entity state, arrives at a conclusion ("the order is probably delivered"), and then acts on that conclusion as if it were a committed fact written to the system of record.

## Bad example

```
User: The customer says the package arrived. Should we issue the refund?

Agent: Based on the customer's message, the order appears to be delivered.
       I'll go ahead and process the refund.

[calls refundPayment(order_id="ord-8821")]
```

The agent estimated a state from user input and immediately acted on that estimate. No verification. No check against the system of record.

## Why it fails

There are two fundamentally different types of state in any agent system:

1. **Believed state (NWM):** What the agent understands from conversation, documents, or reasoning. This is a model — it can be wrong, incomplete, or outdated.
2. **Committed state (SWM):** What the system of record has durably recorded. This is the source of truth.

When you conflate these, you get:
- Refunds issued for orders that were already refunded (refund_status was REFUNDED in the DB, but the agent didn't check)
- Actions taken on entities that don't exist
- State transitions that violate business rules because the agent didn't read the actual current state
- Audit trails that don't reflect what actually happened in the system

The LLM is an estimator. It is very good at reasoning. It is not a database. It cannot know committed state unless you give it committed state.

## World-model fix

Clearly separate the two models:

- **NWM (Narrative World Model):** The LLM's working understanding. It can propose actions. It can summarize context. It cannot authorize them.
- **SWM (Structural World Model):** The system of record. It adjudicates every proposed action against committed state before anything executes.

The flow is: NWM proposes → SWM reads committed state → guards evaluate → transition fires (or doesn't).

The LLM never bypasses this. Committed state is always read fresh from the source of truth at decision time.

## ESTC lens

**Entity:** `Order`  
**State:** Must be read from SWM (database, event log, state machine) — not inferred by NWM  
**Transition:** Only fires after SWM confirms current state matches `from` state  
**Constraint:** Guards run against SWM data, not NWM estimates  

## Good example

```yaml
agent_proposal:
  action: RequestRefund
  entity: Order
  id: "ord-8821"
  basis: "Customer reports delivery confirmed"

swm_check:
  committed_state:
    status: DELIVERED
    delivered_at: "2026-04-20T14:32:00Z"
    refund_status: null
  transition_valid: true
  guards_passed: [refund_window_open, item_is_refundable]

verdict:
  decision: ALLOW
  committed_by: SWM
  note: "NWM proposal validated against committed state"
```

The agent proposed. The SWM verified. The transition was allowed only after committed state was confirmed.

## Takeaway

The LLM can propose. Only the system of record can commit.
