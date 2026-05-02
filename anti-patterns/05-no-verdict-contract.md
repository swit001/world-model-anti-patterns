# Anti-Pattern 05: No Verdict Contract

## Symptom

The agent returns natural language conclusions — "Looks good", "Done", "This seems fine", "I went ahead and processed it" — instead of a structured outcome that downstream systems can act on reliably.

## Bad example

```
User: Should we approve this refund request?

Agent: I reviewed the order. The customer has been with us for two years and 
       the item is within the return window. I think we should go ahead and 
       approve it. Looks good to me!
```

What does "Looks good to me" mean to the system? Nothing. It's prose. The calling code has no idea whether to approve, deny, escalate, or retry. It has to parse intent from text — and that will fail.

## Why it fails

Agents that return prose create a class of problems:

- **Ambiguous outcomes:** "Looks good" could mean ALLOW, or it could mean "I didn't find anything obviously wrong." These are different.
- **Unparseable results:** Downstream systems cannot branch on natural language reliably. String matching on "yes"/"no" breaks on paraphrases.
- **Missing next steps:** There's no `next_state`, no `failed_guard`, no `escalation_reason`. The caller has no structured path forward.
- **Auditability gaps:** You cannot build a reliable audit trail from prose. You need structured decisions with timestamps, reasons, and outcomes.
- **Silent failures:** "I went ahead and processed it" is not a confirmation. Did the tool call succeed? Was state updated? You don't know.

## World-model fix

Define a `VerdictOutcome` contract. Every agent decision must return a structured result with a machine-readable decision, the reason for it, and enough context to take the next action.

The three valid verdicts are a good starting point:
- `ALLOW` — the action is permitted; transition should proceed
- `DENY` — the action is not permitted; transition is blocked
- `ESCALATE` — the system cannot resolve this; route to a human or a higher-authority process

## ESTC lens

**Entity:** `Order`  
**State:** `next_state` is part of the verdict contract — the caller knows what state to write  
**Transition:** The verdict names the attempted transition so the audit log is unambiguous  
**Constraint:** `failed_guard` is included so the denial reason is explicit and actionable  

## Good example

```json
{
  "verdict": "DENY",
  "entity": "Order",
  "entity_id": "ord-8821",
  "attempted_transition": "RequestRefund",
  "current_state": "DELIVERED",
  "next_state": null,
  "failed_guard": "refund_window_open",
  "reason": "Refund window expired. Order delivered 2026-04-10, window closed 2026-04-17.",
  "escalation_path": null,
  "timestamp": "2026-05-02T09:14:00Z"
}
```

The calling system receives a structured object. It knows the decision, the reason, which guard failed, and that no state transition should be written. No text parsing required.

## Takeaway

If your agent's answer can't be parsed by a switch statement, it's not a verdict — it's a guess.

## Examples

- Bad: [examples/bad/no-verdict-contract.md](../examples/bad/no-verdict-contract.md)
- Good: [examples/good/verdict-contract.json](../examples/good/verdict-contract.json)
