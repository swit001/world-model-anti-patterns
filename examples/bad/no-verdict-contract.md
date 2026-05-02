# Bad Example: No Verdict Contract

This is what it looks like when an agent returns natural language conclusions instead of a structured outcome. Downstream systems cannot reliably parse, route, or audit prose responses.

---

## Example 1: Ambiguous resolution

```
User: Should we approve this refund request for order #8821?

Agent: Done, I handled it.
```

Did the agent approve the refund? Deny it? Escalate it? Call a tool successfully? There is no way to know. The calling system has nothing actionable to act on.

---

## Example 2: Confident but unstructured

```
User: Process the refund for order #9002.

Agent: Looks good. The order was delivered and the item is eligible.
       I went ahead and approved it.
```

This reads as confident and complete. But:
- Was the tool call successful?
- What is the new state of the order?
- Which guards were evaluated?
- Is there an audit record?
- What should the caller do next?

None of these questions have answers. The response contains no structured data — only prose.

---

## What's wrong here

Agents that return prose create a class of failures:

- **Ambiguous outcomes:** "Looks good" could mean ALLOW, or it could mean "I didn't see anything obviously wrong." These are different decisions with different consequences.
- **Unparseable results:** Downstream systems cannot reliably branch on natural language. String-matching on "yes" / "approved" / "done" breaks on paraphrases.
- **Missing next-state:** There is no `next_state` field. The caller does not know what to write to the system of record.
- **No audit trail:** You cannot build a reliable audit log from prose. Structured decisions require structured records.
- **Silent failures:** "I went ahead and processed it" is not a confirmation. The tool call may have failed. The state may not have been updated.

---

## The key failure

```
"Done, I handled it."
```

A downstream system cannot route on this. A compliance audit cannot use this. An on-call engineer debugging at 2am cannot trust this.

---

See [anti-patterns/05-no-verdict-contract.md](../../anti-patterns/05-no-verdict-contract.md) for the full analysis and fix.
