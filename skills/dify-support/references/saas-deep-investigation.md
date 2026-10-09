# SaaS Deep Investigation

Use this reference when a SaaS ticket needs root-cause investigation beyond a policy/doc answer. The goal is to reconstruct the real execution chain and verify each hop with the strongest read-only evidence available.

## Contents

- Default flow and entry-to-evidence map
- Source, SigNoz, and DB patterns
- Evidence ledger and causal model
- Escalation and stop criteria
- Shallow-investigation anti-patterns

## Default Flow

1. Start from live Zendesk: latest customer message, screenshots, links, timestamps, requester email, workspace/app/run IDs, and previous agent replies.
2. Normalize the symptom: where it happens, what should happen, what actually happened, and whether the customer asks for explanation, fix, refund, or confirmation.
3. Determine whether the evidence supports object-specific, tenant-specific, multi-tenant/platform, or unknown impact; use cross-object aggregation and a comparable baseline rather than the ticket count alone.
4. Build the real execution chain from the product entry to the final error/decision.
5. Use aligned source code to confirm handler/service/runtime path when the behavior is product logic, not just state.
6. Query the right evidence for each hop through OpenConnector: DB for persisted state, SigNoz for runtime behavior, billing/Stripe records for payment/subscription timeline, plugin DB/runtime for plugin issues, monitoring for infrastructure state, and `cloudflare_mcp/default` read operations only when the edge path is implicated.
7. Record every material claim in the `SKILL.md` evidence ledger with identifiers, time window, exact result, causal role, and status.
8. Keep `unknown` explicit for any hop or causal role that cannot be verified with current read-only tools.
9. Draft only after the ledger explains both the customer-visible behavior and the current state.

## Entry To Evidence Map

| Entry or symptom | Must verify |
| --- | --- |
| WebApp / published app | app status, conversation/message/run records, frontend/public API route, SigNoz API logs/traces, Cloudflare only if access/CDN/WAF suspected |
| Console / Studio | workspace/app permissions, UI API route, source handler/service, DB state, SigNoz API logs |
| Workflow preview/run | workflow_runs, workflow_node_executions, app worker/Celery traces, node-specific source, provider/plugin/runtime logs |
| Dataset indexing | documents/segments/indexing status, queue/Celery publish and consume, worker errors, vector backend/TiDB state when storage is involved |
| Plugin install/invoke | plugin DB installation state, installed plugin version, marketplace package, plugin daemon metrics/logs, Lambda/runtime mapping when relevant |
| Model/provider failure | installed provider/plugin version, model config, provider API error, SigNoz trace/log, source path for error wrapping |
| Billing/quota/rate limit | billing DB, Stripe/payment/webhook timeline, runtime quota/cache state, reserve rejection logs, post-recovery successful runs |
| Edge/CDN/WAF/DNS/TLS | hostname/zone, path/method/status, exact UTC window, region/colo and CF-Ray when available; OpenConnector `cloudflare_mcp/default` evidence correlated with the matching SigNoz origin hop |
| 5xx/timeout/slowness | SigNoz trace first, then logs, then pod/host/K8s metrics, queue/worker state, Cloudflare if edge/origin is implicated |

## Source Tracing

- Align source with the real SaaS deployed version or installed plugin/provider version before using code as evidence.
- Use source code to discover the full execution chain whenever the behavior depends on product logic. Start from route/API path/error text and follow handler -> service -> domain logic -> state/runtime/external dependency.
- Search by concrete anchors first: API path, error text, feature_key, node type, run ID field name, provider/plugin name, class/function from traceback.
- Follow handler -> service -> runtime check -> persistence/cache/external call. Stop reading only after finding where the decision or error is produced.
- Source is the map, not the proof. It proves logic and possible/current paths; it does not prove what happened to the customer until DB/SigNoz/runtime/billing evidence matches the same identifiers and time window.

## SigNoz Pattern

- Start narrow: tenant/workspace/app/run/request/trace/time window.
- For scope, aggregate the same error/latency signal across tenants, deployments, operations, or execution pools and compare the incident window with a nearby baseline.
- If no result, verify service/deployment names and table freshness before saying no logs.
- Prefer structured service/field discovery, aggregation, search, trace details, and metric tools. Use raw `sz_run` only for advanced dimensions the structured surface cannot express.
- For async work, connect producer and consumer by `messaging.message.id`, Celery task name, queue/destination, trace ID, or correlated timestamp.
- For slowness/timeouts, trace first to locate the slow boundary, then metrics/logs for resource or dependency evidence.
- Avoid broad multi-day fuzzy scans; narrow by time and identifier, then widen only if needed.

## DB Pattern

- Use DB to confirm persisted state and object ownership, not as the whole root cause.
- For any field/table uncertainty, query `information_schema` live.
- For live-state-dependent issues, compare state at customer error time when possible and current state after fix/retry.
- Cross-check related records: app -> workflow -> run -> node execution; dataset -> document -> segments; workspace -> subscription -> feature usage; plugin installation -> runtime/logs.

## Evidence And Causality

- Attach every conclusion to the correct customer object, entry path, version, and time window. A tool or table name alone is not evidence.
- Keep symptom, first proven failing boundary, immediate trigger, root cause, contributing factors, recovery event, and current state separate.
- A confirmed error boundary is not automatically a confirmed root cause.
- Cross-tenant impact is not itself a root cause. Co-occurrence in one time window establishes correlation only until both symptoms are tied to the same dependency, capacity event, deployment, limit, trace, or recovery event.
- Treat prior tickets, common playbooks, screenshots without context, current-only snapshots, and source branches not proven for the customer as hypotheses or supporting context, not facts.
- Treat an empty search as negative evidence only after verifying service/deployment names, schema, table freshness, retention, time conversion, identifiers, and alternate correlation keys.
- For recovered incidents, prove the historical failure and a matching post-recovery success separately. Mark the recovery event `unknown` if the changing state is not visible.

## Escalation Question

When the remaining gap is owner/SRE-only, provide a precise ask:

```text
Please help confirm <tenant/app/run/subscription> during <time window>.
Verified so far: <Zendesk/DB/SigNoz/source/billing facts>.
Missing hop: <Redis/cache/runtime quota/webhook/cron/queue/provider state>.
Need: <exact key, endpoint, event ID, job name, trace, or expected log>.
Impact: <customer-visible error and current state>.
```

## Stop Criteria

Stop only when:

- the completed evidence ledger supports the causal chain and current state; or
- the case is fully answered by customer-safe policy/docs plus current facts; or
- the exact remaining owner-only evidence is named and the handoff question is ready.

## Shallow Investigation Anti-Patterns

Do not end with these:

- "DB is normal now" without explaining the customer error window.
- "No SigNoz error found" without verifying service/deployment names, time range, table freshness, and alternative trace/log keys.
- "Looks like provider/plugin issue" without checking installed version, runtime logs, and whether multiple tenants/plugins are affected.
- "Workflow failed" without checking node executions, worker traces, provider/plugin call, message/conversation metadata, and preview vs published path.
- "Customer needs to provide more information" before using available email, workspace, link, screenshot, timestamp, run ID, error text, and Zendesk attachments.
- "Known issue" without validating the current runtime path and whether the fix is already deployed.
- "Recovered" without checking both current effective state and a successful post-recovery action.
- A confirmed boundary error presented as the root cause without evidence for why that boundary failed.

If one of these is tempting, continue the chain or mark the missing hop explicitly.
