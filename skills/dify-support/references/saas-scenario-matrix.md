# SaaS Scenario Matrix

Use this reference for non-trivial SaaS tickets. Pick every relevant row; real tickets often cross modules. The goal is coverage: entry, source path, persisted state, runtime evidence, external dependency, and current recovery.

## Contents

- Cross-cutting intake checklist
- Scenario rows and common misses
- Evidence-ledger output shape
- Shallow-investigation red flags

## Cross-Cutting Checklist

For any SaaS issue, capture:

- customer timestamp and timezone, then normalize to UTC for logs;
- requester email, workspace ID, app/workflow/dataset/document/conversation/run IDs;
- exact UI path/API path/WebApp URL and screenshot/error text;
- whether this is preview/debug, published app, API, embedded widget, or console;
- for a suspected edge path, hostname/zone, path/method/status, exact UTC window, customer region or colo, and CF-Ray when available;
- whether the issue is current, historical, intermittent, or already recovered;
- whether scope is object-specific, tenant-specific, multi-tenant/platform, or still unknown, including the comparison window used;
- whether prior agent replies introduced assumptions that need verification.

Do not stop until each applicable row below is either checked or explicitly marked not relevant.

Use aligned source code as the default way to discover the full chain when the symptom depends on product behavior. Then use DB, SigNoz, billing, runtime/cache, queue, and provider evidence to prove which branch the customer actually hit.

## Scenario Rows

| Scenario | Anchors | Must Check | Common Miss |
| --- | --- | --- | --- |
| WebApp / published app access | public app URL, app_id, browser, region, status code | app status, app mode, published config, public API route, Cloudflare/WAF if access fails, SigNoz API logs, frontend setting source | Checking console app state but not published app state |
| Edge / CDN / WAF / DNS / TLS | hostname/zone, path/method/status, UTC window, region/colo, CF-Ray | OpenConnector `cloudflare_mcp/default` company account/zone, applicable HTTP analytics, security events, DNS/TLS state or Workers observability, matching SigNoz origin evidence | Treating an empty result as proof before checking scope, permissions, retention, sampling, and origin evidence; using Wrangler, direct MCP, or dashboard as fallback |
| Console / Studio UI error | route, screenshot, workspace/app ID, user email | requester membership/role, workspace status, UI API endpoint, backend source handler, DB state, SigNoz logs | Treating frontend display as source of truth |
| API failure | endpoint, API key type, status code, request_id, app/workspace ID | API key scope, app API enabled, route/source handler, DB object ownership, SigNoz trace/log, rate limit/quota | Not distinguishing app API key vs workspace key |
| Workflow preview/debug | workflow_id, run_id, node type, preview/published | workflow_runs, node executions, app worker/Celery spans, node source, provider/plugin call, debug-only path differences | Assuming preview path equals published app path |
| Published workflow/chatflow run | app_id, conversation_id, message_id, workflow_run_id | app mode, published workflow version, conversation/message records, workflow_runs, node executions, SigNoz API+worker traces | Checking draft workflow instead of published version |
| Annotation reply / marked answer | message text, annotation hit, metadata, workflow_run_id | annotation match records, message metadata, response path source, whether workflow should short-circuit, preview SSE/status close behavior, logs around completion | Looking only at returned answer while missing unfinished preview state |
| Dataset indexing / document stuck | dataset/document IDs, indexing status, upload time | documents, segments, indexing tasks, Celery producer/consumer, worker errors, parser/extractor path, vector backend/TiDB if storage involved | Reading document row but not queue/worker consumption |
| Retrieval/RAG quality | query, dataset IDs, retrieval node, expected doc | dataset/doc/segment state, embedding/rerank config, retrieval logs/traces, source path for filters, model/provider version | Treating quality complaint as purely prompt issue |
| Model/provider error | provider, model, plugin version, exact provider error | workspace model config, installed provider plugin, marketplace/latest version, source error wrapping, provider status/error, SigNoz trace | Saying Dify issue without verifying provider/plugin version |
| Plugin install/invoke | plugin identifier/version, tenant, tool call, error | plugin installation DB, runtime type, marketplace package, plugin daemon logs/metrics, Lambda/serverless mapping if relevant, source version | Calling all plugin errors third-party without proving platform/runtime boundary |
| MCP/tool connector | tool_installation ID, endpoint, protocol, timeout | plugin DB, endpoint/protocol config, API/plugin daemon traces, network/timeout boundary, source path | Missing protocol/path mismatch such as SSE vs MCP |
| Billing/subscription | tenant, Stripe customer/subscription/invoice, billing page time | billing DB, Stripe/payment/webhook timeline, subscription status transitions, cron/sync logs, current effective state | Checking only current plan |
| Quota/rate limit/workflow reserve | feature_key, error, run/API time, tenant | billing feature config, runtime quota/cache/Redis or quota info, reserve/check logs, workflow/API rejection, post-recovery success | Using `tenant_feature_usages.usage` as realtime truth |
| Credits / AI usage | tenant, billing period, model calls, credit pool | credit pools, credit logs, model usage if available, subscription plan, reset time, unusual loops/runs | Confusing AI credits with API/workflow quota |
| Knowledge storage / vector quota | tenant, dataset/doc IDs, storage UI number | plan quota, TiDB/vector cluster mapping, document/segment counts, storage metrics, quota cache if enforcement mismatch | Treating Postgres quota limit as used storage |
| File upload / audio / large payload | file size/type, endpoint/tool, 413/timeout | gateway/API limit, plugin/tool path, object storage, parser/worker logs, Cloudflare if edge rejects | Not locating which layer rejected the upload |
| Auth / permission / team role | requester email, workspace, action, role | account, tenant joins, role/permission checks, workspace status, source authorization path, audit/log evidence | Assuming requester owns workspace |
| SSO / domain / login | email domain, login method, org/domain, error | identity records, workspace membership, auth provider path, logs, current product capability | Mixing Enterprise SSO assumptions into SaaS |
| Conversation/message anomaly | conversation_id, message_id, input/output, UI state | messages, message annotations/metadata, workflow_run_id, response mode/SSE, API logs, frontend close/status behavior | Checking workflow_runs only and ignoring message metadata |
| Performance/timeout/intermittent | time window, endpoint/run/node, user region | trace waterfall, service dependency, worker queue, K8s/pod metrics, provider/plugin latency, Cloudflare if edge path | Broad log grep without trace boundary |
| Platform incident / many customers | time window, customer anchor, symptom | prove customer sample, aggregate affected tenants/objects/execution pools, compare baseline, SigNoz service/error/latency and exposed K8s/queue metrics, recent deployment evidence; use Cloudflare only when the edge path is implicated | Inferring platform scope from one ticket, or common root cause from timing alone |
| Recent regression / product bug | "worked before", date, version, exact behavior | deployed image tag, source diff, GitHub issues/PRs, logs proving path, compare old/new behavior | Reading current code without checking recent change |

## Evidence-Ledger Output Shape

For internal analysis, keep this structure:

```text
Scenario rows checked:
Entry and customer symptom:
Expected vs actual:
Execution chain:
Evidence ledger:
- Claim:
  Causal role:
  Status:
  Evidence source:
  Object identifiers:
  Time window and timezone:
  Exact observed result:
  Alternative explanations checked:
First proven failing boundary:
Immediate trigger:
Root cause:
Contributing factors:
Recovery event:
Ruled out:
Unknown / owner-only gaps:
Current state:
Customer-safe conclusion:
```

## Red Flags

These are signs the investigation is too shallow:

- conclusion based on one DB query;
- no source path for a product-logic claim;
- no SigNoz/runtime evidence for a runtime failure;
- no historical timeline for a recovered issue;
- no current-state check after saying fixed;
- customer asked "why did it happen" but answer only says "now it is normal";
- workflow issue checked only `workflow_runs`, not node execution, message metadata, worker logs, and published/preview path;
- plugin issue checked only installation DB, not plugin daemon/runtime logs;
- billing/quota issue checked only plan, not payment/webhook/runtime quota;
- customer was asked for more data before using available workspace/email/link/error/time anchors.
- a proven error boundary was labeled as the root cause without evidence for its underlying cause.
