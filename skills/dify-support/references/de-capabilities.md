# DE Capability Gaps

Use this reference when a support investigation needs a data source or platform permission that the current support toolchain does not expose.

## Available

- Read-only DB queries for `prod`, `billing`, and `plugin` through `dify_db` MCP.
- SigNoz MCP for logs, tracebacks, services, schema, and ClickHouse SQL.
- Redis telemetry through SigNoz trace spans when instrumented: service, command family, latency, errors, status message, and peer host. Redis keys and values are normally masked.
- Zendesk MCP for ticket, customer, attachment, article, view, and metadata access.
- Exposed SigNoz metrics for request success rate, Kubernetes state, Celery queues, and plugin daemon metrics.

## Partially Available

- Celery queue visibility exists only where the remote SigNoz MCP exposes matching evidence; queue mutation or retry is an SRE action.
- Plugin daemon metrics exist, but plugin runtime recovery actions are owner/SRE actions.
- Trace views should be queried through SigNoz MCP where possible; if a specific view is not exposed, record the missing evidence channel instead of guessing.

## Missing Or Owner-Only

- Redis/runtime quota key values and cache counters, including quota `q/u/r` style keys. SigNoz can show Redis spans, but not the exact key contents.
- Stripe Dashboard writes and payment operations.
- Cron replay, webhook replay, queue retry, pod restart, database writes, or any production mutation.
- Sandbox disk/inode monitoring if not exposed through current metrics.

When one of these is required to close the chain, ask SRE/owner a precise question: final state source, key or endpoint, event/job identifier, timestamp window, and expected evidence.
