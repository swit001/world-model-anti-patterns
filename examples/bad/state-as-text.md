# Bad Example: State as Text

This is what it looks like when state lives entirely in a prompt. The agent must infer what is true from natural language — no structured state, no source-of-truth check.

---

## Prompt

```
System: You are a customer service agent with the ability to issue refunds.
        Use the information provided to decide whether to approve refund requests.

User: Here is the order summary for order #8821:

  - Customer placed order on April 18
  - Order shipped April 19 via FedEx
  - Tracking shows "Delivered - Front Door" on April 20
  - Customer emailed on May 1 saying the item was defective

  Should we refund this order?
```

## Agent response

```
Based on the order summary, the item was delivered on April 20 and the customer 
has reported it as defective. The order appears to be in a delivered state and 
the refund request seems legitimate. I'll go ahead and issue the refund.

[calls refundPayment(order_id="ord-8821", amount=49.99)]
```

---

## What's wrong here

The agent:

1. Read a text summary — not a structured state field
2. Inferred `status: DELIVERED` from a FedEx tracking note
3. Did not check whether a refund had already been issued
4. Did not verify the refund window was still open
5. Did not confirm the item was marked as refundable in the catalog
6. Acted immediately on its inference

The system of record may tell a completely different story. The order may have been partially refunded already. The item may be marked non-refundable. The refund window may have closed. None of this was checked.

---

## The key failure

```
"The order appears to be in a delivered state"
```

"Appears to be" is not `status: DELIVERED`. One is a belief. The other is a fact.

---

See [anti-patterns/01-state-as-text.md](../../anti-patterns/01-state-as-text.md) for the full analysis and fix.
