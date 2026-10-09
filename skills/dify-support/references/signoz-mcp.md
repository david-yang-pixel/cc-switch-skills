# OpenConnector SigNoz Reference

Use this reference when a SaaS support case needs runtime evidence from logs, traces, queues, workers, host metrics, Kubernetes metrics, or ClickHouse SQL.

## Contents

- Required path and investigation routing
- Commands and ClickHouse tables
- Logs, traces, and Redis telemetry
- Celery, messaging, and Kafka
- Host and Kubernetes metrics
- Query guardrails

## Required path

- Use only OpenConnector `custom_mcp` provider, connection `dify-signoz`.
- Inspect the current action surface through `custom_mcp.list_tools`; do not cache the tool count or schema. An existing task may hold an older surface until MCP reload.
- Prefer structured tools for discovery, aggregation, search, trace details, and metrics. Use `sz_run` only when the structured tools cannot express the query.
- For `sz_run`, pass commands as `{"args":[...]}`. Raw SQL is restricted to one read-only query.
- If the connection, authentication, policy, or required action is unavailable, record that exact OpenConnector hop and continue other evidence routes. Do not call the remote endpoint directly.
- Do not use local `sz`, `/app/scripts/sz`, old `signoz-mcp-server`, or the SigNoz Web UI as fallback for this support workflow.
- Keep all queries read-only.

## Investigation routing

| Symptom | First SigNoz path |
| --- | --- |
| API 500 / UI error / plugin error | search logs/spans -> exact trace details -> raw traceback only if needed |
| Workflow/RAG/indexing stuck | DB state -> trace search -> exact trace -> raw Celery/messaging SQL -> pod metrics |
| Waiting/running state unexplained | DB state -> field discovery for message/run keys -> Producer/Consumer spans |
| Slow/timeout/intermittent | trace aggregation -> slow trace details -> dependency logs -> pod metrics |
| Multi-user/systemic impact | aggregate by tenant/deployment/operation -> baseline comparison -> pod/capacity metrics |
| Suspected Kafka | narrow by topic/service/trace/time; avoid broad fuzzy scans |

## Structured tool selection

| Need | Tool |
| --- | --- |
| Resolve active service/deployment names | `signoz_list_services` |
| Discover actual attribute/resource keys | `signoz_get_field_keys` |
| Discover valid values before filtering | `signoz_get_field_values` |
| Retrieve matching log records | `signoz_search_logs` |
| Count/group logs or build a time series | `signoz_aggregate_logs` |
| Find spans by service, operation, tenant, error, or duration | `signoz_search_traces` |
| Compare count/P50/P95/P99 across tenants, operations, or deployments | `signoz_aggregate_traces` |
| Reconstruct one trace | `signoz_get_trace_details` |
| Discover metric names | `signoz_list_metrics` |
| Query Pod/deployment metrics | `signoz_query_metrics` |
| Advanced Celery/Kafka/Redis/dependency/custom SQL | `sz_run` |

Use relative `timeRange` values such as `30m`, `2h`, or `1d`. For grouped attribute/resource fields, use explicit contexts such as `attribute.dify.tenant_id` and `resource.k8s.deployment.name`. Discover keys/values instead of guessing when a filter returns zero rows.

Structured tools currently accept relative windows, not arbitrary absolute start/end timestamps. For an older incident that requires exact UTC subwindows, two non-overlapping baseline windows, custom filters, cumulative duration, concurrency/slot approximations, or a dimension not present in the schema, use one targeted read-only `sz_run query` with explicit time predicates and `LIMIT`.

## Impact-scope sequence

1. Prove the customer symptom with the exact tenant/run/operation and UTC window.
2. Aggregate the same signal by tenant, deployment, execution pool, service, or operation inside the incident window.
3. Repeat the identical aggregation for a nearby baseline window.
4. Compare contributor concentration: top tenants/apps by call count, errors, cumulative duration, average/P95 duration, and execution pool.
5. Record affected tenant/object counts, rate or percentile change, and execution pools involved.
6. Query metrics for the implicated deployment/Pod only after the runtime boundary is identified.
7. Keep impact scope separate from load origin: one tenant may create pressure that affects many tenants.
8. Treat co-occurring symptoms as correlated, not same-root-cause, until a shared dependency, capacity event, deployment, limit, trace, or recovery event connects them.

Environment is a preferred scope, not an assumption that overrides evidence. If an environment-filtered query returns zero rows, check field keys/values and signal freshness; retry with a narrow deployment/service/cluster and time window before treating the result as negative evidence.

## Raw `sz_run` commands

```json
{"args":["services","--env","production","--hours","1","--limit","30"]}
```

```json
{"args":["errors","--env","production","--tenant","<tenant_id>","--hours","24","--limit","50"]}
```

```json
{"args":["errors","--env","production","--deployment","<deployment>","--grep","<substring>","--hours","6","--limit","20"]}
```

```json
{"args":["traceback","--id","<log_id>","--hours","24","--limit","5"]}
```

`--help` returns non-JSON text and may fail through the MCP wrapper. Use the fixed subcommands above.

## Main ClickHouse tables

| Area | Table | Notes |
| --- | --- | --- |
| Logs | `signoz_logs.distributed_logs_v2` | `timestamp` is UInt64 ns; use `{start_ns}` / `{end_ns}` |
| Trace spans | `signoz_traces.distributed_signoz_index_v3` | service, span, duration, status, HTTP, DB, messaging, errors |
| Trace summary | `signoz_traces.distributed_trace_summary` | trace id, start/end, span count |
| Error index | `signoz_traces.distributed_signoz_error_index_v2` | exception type/message/stacktrace by trace/span |
| Metrics series labels | `signoz_metrics.distributed_time_series_v4` | metric labels in `attrs` / `resource_attrs` |
| Metrics samples | `signoz_metrics.distributed_samples_v4` | values keyed by `fingerprint` + `metric_name` |
| Metrics 5m/30m aggregates | `distributed_samples_v4_agg_5m`, `distributed_samples_v4_agg_30m` | use for longer windows |

## Logs and traces

Common log fields:

- `body`
- `severity_text`
- `trace_id`, `span_id`
- `attributes_string['tenant_id']`
- `resources_string['k8s.deployment.name']`
- `resources_string['service.name']`
- `resources_string['deployment.environment']`

Common trace fields:

- `serviceName`, `name`, `kind_string`
- `duration_nano`, `has_error`, `status_code_string`, `status_message`
- `http_url`, `http_method`, `httpRoute`
- `dbSystem`, `dbOperation`, `dbName`
- `attributes_string[...]`, `resources_string[...]`

## Redis telemetry

SigNoz can query Redis-related trace spans when Redis calls are instrumented. Use this to prove whether Redis was called, by which service, which command family ran, whether it errored, and latency. It is not a Redis data reader: Redis keys and values are usually masked in `attributes_string['db.statement']` as `GET ?`, `SETEX ? ? ?`, `PUBLISH ? ?`, etc. Exact runtime quota/cache key values remain an owner/SRE or dedicated Redis read-only-tool requirement.

Redis span overview:

```json
{"args":["query","SELECT serviceName, name, count() AS spans, countIf(has_error) AS errors, round(quantile(0.95)(duration_nano)/1000000, 2) AS p95_ms FROM signoz_traces.distributed_signoz_index_v3 WHERE timestamp >= fromUnixTimestamp64Nano({start_ns}) AND timestamp <= fromUnixTimestamp64Nano({end_ns}) AND dbSystem = 'redis' GROUP BY serviceName, name ORDER BY spans DESC LIMIT 50","--hours","1"]}
```

Redis error overview:

```json
{"args":["query","SELECT serviceName, name, count() AS errors, any(status_message) AS sample_status FROM signoz_traces.distributed_signoz_index_v3 WHERE timestamp >= fromUnixTimestamp64Nano({start_ns}) AND timestamp <= fromUnixTimestamp64Nano({end_ns}) AND dbSystem = 'redis' AND has_error GROUP BY serviceName, name ORDER BY errors DESC LIMIT 30","--hours","1"]}
```

Sample Redis attributes:

```json
{"args":["query","SELECT serviceName, name, mapKeys(attributes_string) AS attr_keys, attributes_string['db.statement'] AS db_statement, attributes_string['net.peer.name'] AS peer, status_message FROM signoz_traces.distributed_signoz_index_v3 WHERE timestamp >= fromUnixTimestamp64Nano({start_ns}) AND timestamp <= fromUnixTimestamp64Nano({end_ns}) AND dbSystem = 'redis' LIMIT 20","--hours","1"]}
```

Do not run broad fuzzy scans over `attributes_string` for Redis. Prefer `dbSystem = 'redis'` plus a short time window, `serviceName`, command `name`, `trace_id`, tenant/run/log-derived timestamp, or the concrete service involved in the execution chain.

Slow/error span aggregation:

```json
{"args":["query","SELECT serviceName, name, count() AS spans, countIf(has_error) AS errors, round(quantile(0.95)(duration_nano)/1000000, 2) AS p95_ms FROM signoz_traces.distributed_signoz_index_v3 WHERE timestamp >= fromUnixTimestamp64Nano({start_ns}) AND timestamp <= fromUnixTimestamp64Nano({end_ns}) GROUP BY serviceName, name ORDER BY errors DESC, p95_ms DESC LIMIT 20","--hours","1"]}
```

## Celery and messaging

Celery spans are in `signoz_traces.distributed_signoz_index_v3`. Key attributes:

- `attributes_string['celery.action']`: usually `apply_async` or `run`
- `attributes_string['celery.task_name']`
- `attributes_string['messaging.message.id']`
- `attributes_string['messaging.destination']`
- `attributes_string['messaging.destination_kind']`
- `attributes_string['celery.hostname']`
- `attributes_string['celery.state']`
- `attributes_string['exception.type']`

Queue/task overview:

```json
{"args":["query","SELECT attributes_string['messaging.destination'] AS queue, attributes_string['celery.task_name'] AS task, kind_string, serviceName, count() AS spans, countIf(has_error) AS errors, round(100*errors/spans,3) AS error_pct, round(quantile(0.95)(duration_nano)/1000000,2) AS p95_ms FROM signoz_traces.distributed_signoz_index_v3 WHERE timestamp >= fromUnixTimestamp64Nano({start_ns}) AND timestamp <= fromUnixTimestamp64Nano({end_ns}) AND mapContains(attributes_string, 'celery.task_name') GROUP BY queue, task, kind_string, serviceName ORDER BY spans DESC LIMIT 80","--hours","1"]}
```

Trace one Celery task by message id:

```json
{"args":["query","SELECT timestamp, serviceName, name, kind_string, has_error, duration_nano, attributes_string['celery.action'] AS action, attributes_string['celery.state'] AS state, attributes_string['messaging.destination'] AS queue, attributes_string['messaging.message.id'] AS message_id, attributes_string['celery.hostname'] AS hostname FROM signoz_traces.distributed_signoz_index_v3 WHERE timestamp >= fromUnixTimestamp64Nano({start_ns}) AND timestamp <= fromUnixTimestamp64Nano({end_ns}) AND attributes_string['messaging.message.id'] = '<message_id>' ORDER BY timestamp ASC LIMIT 20","--hours","24"]}
```

Interpretation:

- Producer span exists, no Consumer span: task was published but not observed as consumed in the window.
- Producer and Consumer exist, Consumer `SUCCESS`: queue path worked; inspect downstream span/log if user still sees failure.
- Consumer `FAILURE` or `has_error=1`: use `trace_id`, `span_id`, `status_message`, and `exception.type`.
- No Producer span: task may not have been created; check DB/business state and API logs.

## Kafka

Kafka is queryable only when instrumentation/logs exist for the relevant service/topic/time range. Prefer a concrete topic, service, trace id, or short time window.

```json
{"args":["query","SELECT serviceName, name, kind_string, attributes_string['messaging.destination'] AS topic, count() AS spans, countIf(has_error) AS errors FROM signoz_traces.distributed_signoz_index_v3 WHERE timestamp >= fromUnixTimestamp64Nano({start_ns}) AND timestamp <= fromUnixTimestamp64Nano({end_ns}) AND (lower(name) LIKE '%kafka%' OR lower(attributes_string['messaging.system']) LIKE '%kafka%' OR attributes_string['messaging.destination'] = '<topic>') GROUP BY serviceName, name, kind_string, topic ORDER BY spans DESC LIMIT 50","--hours","1"]}
```

Avoid 7-day fuzzy scans over `attributes_string` or log `body`; they can hit ClickHouse memory limits or time out. If no Kafka spans/logs appear in the short window, state that SigNoz has no observed Kafka evidence for that scope rather than guessing.

## Host and Kubernetes metrics

Metrics are not in the logs/traces tables. Join `distributed_samples_v4` to `distributed_time_series_v4` on `fingerprint` and `metric_name`.

Common host metrics:

| UI column | Metric |
| --- | --- |
| CPU usage | `system.cpu.utilization`, aggregate all non-`idle` states per CPU then average per host |
| Memory usage | `system.memory.utilization` with `attrs['state']='used'` |
| iowait | `system.cpu.utilization` with `attrs['state']='wait'` |
| Load average | `system.cpu.load_average.1m`, `.5m`, `.15m` |
| Host status | latest sample time from a host metric |

Common Kubernetes metrics:

| UI column / object | Metric |
| --- | --- |
| Pod CPU cores | `k8s.pod.cpu.usage` |
| Pod CPU request / limit usage | `k8s.pod.cpu_request_utilization`, `k8s.pod.cpu_limit_utilization` |
| Pod memory WSS | `k8s.pod.memory.working_set` |
| Pod memory request / limit usage | `k8s.pod.memory_request_utilization`, `k8s.pod.memory_limit_utilization` |
| Container ready / restarts | `k8s.container.ready`, `k8s.container.restarts` |
| Pod phase / reason | `k8s.pod.phase`, `k8s.pod.status_reason` |
| Deployment availability | `k8s.deployment.available`, `k8s.deployment.desired` |
| Node conditions | `k8s.node.condition_ready`, `k8s.node.condition_memory_pressure`, `k8s.node.condition_disk_pressure` |

Pod resource snapshot:

```json
{"args":["query","WITH latest AS (SELECT t.resource_attrs['k8s.pod.name'] AS pod, t.resource_attrs['k8s.namespace.name'] AS namespace, t.resource_attrs['k8s.node.name'] AS node, t.resource_attrs['k8s.deployment.name'] AS deployment, t.metric_name AS metric, argMax(s.value, s.unix_milli) AS value FROM signoz_metrics.distributed_samples_v4 AS s INNER JOIN signoz_metrics.distributed_time_series_v4 AS t ON s.fingerprint=t.fingerprint AND s.metric_name=t.metric_name WHERE s.unix_milli >= (toUnixTimestamp(now() - INTERVAL 30 MINUTE) * 1000) AND t.resource_attrs['deployment.environment']='production' AND t.metric_name IN ('k8s.pod.cpu.usage','k8s.pod.cpu_request_utilization','k8s.pod.cpu_limit_utilization','k8s.pod.memory.working_set','k8s.pod.memory_request_utilization','k8s.pod.memory_limit_utilization') GROUP BY pod, namespace, node, deployment, metric) SELECT pod, namespace, node, deployment, round(maxIf(value, metric='k8s.pod.cpu.usage'),3) AS cpu_cores, round(100*maxIf(value, metric='k8s.pod.cpu_request_utilization'),1) AS cpu_req_pct, round(100*maxIf(value, metric='k8s.pod.cpu_limit_utilization'),1) AS cpu_limit_pct, round(maxIf(value, metric='k8s.pod.memory.working_set')/1024/1024/1024,2) AS mem_wss_gb, round(100*maxIf(value, metric='k8s.pod.memory_request_utilization'),1) AS mem_req_pct, round(100*maxIf(value, metric='k8s.pod.memory_limit_utilization'),1) AS mem_limit_pct FROM latest GROUP BY pod, namespace, node, deployment ORDER BY cpu_cores DESC LIMIT 20","--hours","0.01"]}
```

Deployment availability:

```json
{"args":["query","SELECT t.resource_attrs['k8s.deployment.name'] AS deployment, t.resource_attrs['k8s.namespace.name'] AS namespace, maxIf(s.value, t.metric_name='k8s.deployment.available') AS available, maxIf(s.value, t.metric_name='k8s.deployment.desired') AS desired FROM signoz_metrics.distributed_samples_v4 AS s INNER JOIN signoz_metrics.distributed_time_series_v4 AS t ON s.fingerprint=t.fingerprint AND s.metric_name=t.metric_name WHERE s.unix_milli >= (toUnixTimestamp(now() - INTERVAL 30 MINUTE) * 1000) AND t.resource_attrs['deployment.environment']='production' AND t.metric_name IN ('k8s.deployment.available','k8s.deployment.desired') GROUP BY deployment, namespace HAVING deployment != '' ORDER BY desired DESC LIMIT 20","--hours","0.01"]}
```

## Query guardrails

- Always constrain time and the smallest useful identifier: tenant, trace id, message id, deployment, pod, host, queue, or topic. Add environment when the signal carries a verified environment field; do not let a missing label hide real data.
- Prefer structured discovery/aggregation/search/trace tools before raw SQL. Within `sz_run`, prefer `errors`, `traceback`, and targeted trace SQL before broad ClickHouse searches.
- For long metric windows, use 5m/30m aggregate tables instead of raw samples.
- Do not paste raw prompt, customer payload, API keys, stacktrace dumps, AWS/DB/infra details, or SigNoz internals into customer-facing replies. Summarize the failure point in safe wording.
- If SigNoz evidence is absent, say what was checked and that no matching evidence was observed in the selected window.
