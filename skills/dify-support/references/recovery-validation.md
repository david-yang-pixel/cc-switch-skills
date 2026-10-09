# Recovery Validation

Use when the customer is blocked or a fix/workaround is proposed. A diagnosis alone is not a completed support outcome.

For every feasible candidate record:

```text
Candidate action:
Prerequisites and authorization:
Mechanism it addresses:
Expected result:
Validation environment/path:
Observed result:
Behavior after restoring original settings:
Tradeoffs and data risk:
Validation level:
```

Validation levels:

- `code-derived`: source suggests the action could affect the path;
- `isolated reproduction`: observed in a separate controlled path;
- `matching customer-path`: observed on the affected customer object/path;
- `current recovery confirmed`: a matching customer action now succeeds and required persistence/output is verified.

Do not claim recovery from HTTP 200 alone, a successful internal workflow record, a merged PR, a deployment report, another tenant, or an isolated test. Verify the customer's final return path and persisted/visible result where relevant.

Do not recommend destructive actions, data deletion, credential rotation, downgrade, or configuration changes without explaining prerequisites, backup/data impact, tradeoffs, and how to validate and restore. A workaround that hides an input or changes a read window does not prove persisted data was deleted or quota/storage released.

If no candidate is validated, state candidates checked, why they failed or remain unverified, and the smallest remaining customer/owner check. Do not invent a workaround to satisfy the gate.
