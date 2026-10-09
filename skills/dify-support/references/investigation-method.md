# Investigation Method

Use for runtime failures, causal claims, reply review, recovered incidents, or high-risk account/billing questions.

## Evidence rule

Treat every cause as a hypothesis until evidence verifies it for the correct customer object, path, version, and time window.

- `confirmed`: direct evidence proves the claim.
- `evidence-supported`: the evidence chain favors the claim and no stronger alternative remains.
- `ruled out`: evidence contradicts the hypothesis for the relevant path/window.
- `unknown`: evidence is absent, inaccessible, stale, contradictory, or from a neighboring path.

Do not turn `unknown` into a root cause. A source path, prior ticket, common pattern, screenshot without context, current snapshot, or empty log is not enough on its own.

## Intake and depth

Identify the customer's real question, expected/actual behavior, exact entry path, timestamp/timezone, error text, and available workspace/app/workflow/dataset/document/conversation/run/request/trace/provider/subscription identifiers.

| Route | Use when | Minimum |
| --- | --- | --- |
| `fast` | Stable product knowledge, current policy/docs, deterministic usage/configuration | Current authoritative source and applicable environment/version |
| `standard` | One object has a deterministic runtime/configuration failure | Latest ticket state, affected object, failing execution hop, material claims and unknowns, feasible recovery checks |
| `deep` | Intermittent failure, latency, 429/5xx, queues, shared-service symptom, regression, recovery incident | Full execution chain, scope aggregation, baseline, contributor concentration, evidence ledger, recovery validation |
| `high-risk` | Billing, refund, entitlement, identity, permissions, security, Admin mutation | Applicable evidence route plus identity, authorization, dry-run/confirmation, and audit gates |

Update the same investigation when the customer replies. Preserve unresolved questions, superseded hypotheses, attempted actions, and the effective published configuration for each test.

## Impact scope

For deep incidents, prove the anchor object/window first; then compare same-tenant objects, cross-tenant/service/deployment signals, and a nearby baseline using the same dimensions. Classify scope as `object-specific`, `tenant-specific`, `multi-tenant/platform`, or `unknown` with sample counts.

Scope is not root cause. One tenant can contribute pressure to a shared pool while impact is multi-tenant; state those separately. Correlation in time does not prove a common cause.

## Evidence ledger

Before material causal/live-state claims on standard, deep, or high-risk routes, record:

```text
Claim:
Causal role: symptom | failing boundary | immediate trigger | root cause | contributing factor | recovery event | current state
Status: confirmed | evidence-supported | ruled out | unknown
Evidence source:
Object identifiers:
Time window and timezone:
Version / provenance / applicable path:
Exact observed result:
Alternative explanations checked:
```

Keep symptom, first failing boundary, trigger, root cause, contributing factor, recovery event, and current state separate. A confirmed error boundary is not automatically a confirmed root cause. Historical failure, recovery event, and current success require separate proof.

## Stop gate

Stop when either the evidence supports the conclusion and current state, authoritative current sources answer a stable product/policy question, or an unavailable/owner-only hop is precisely identified with object, UTC window, verified facts, missing evidence, and requested owner check.

For quota, billing, reserve, cache, webhook, cron, plugin runtime, rate limit, and async cases, never rely on the current snapshot alone. Explain reported-time failure and current state separately.
