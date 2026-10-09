# DB And SigNoz Capabilities

Read before concluding that a queue/runtime fact is unavailable or handing off to SRE.

## Dify DB

OpenConnector `dify-db` exposes read-only SQL through `db_gateway_query` for `prod`, `billing`, and `plugin`.

It can prove persisted object state, configuration, relationships, timestamps, workflow/node records, quota ledgers, subscription mirrors, and plugin installations when the schema contains them. Inspect `information_schema` for uncertain fields.

Persisted `waiting`, `splitting`, `running`, reserve, or retry state does not by itself prove that a live Redis message still exists, a worker currently owns the task, or the customer response completed. `COMMIT` is consumption evidence; `RESERVE` alone must not be counted as final consumption.

## SigNoz

OpenConnector `dify-signoz` exposes captured logs, traces/spans, metrics, and advanced read-only ClickHouse queries. It can investigate Celery/Kafka/Redis/Kubernetes/dependency behavior when the relevant telemetry was actually collected.

Start with structured service/field discovery, aggregation, matching details, and trace reconstruction. Use `sz_run` only for dimensions not exposed by structured tools. Bound every query by UTC time and result limits.

An interface can show Redis command family, peer, latency, error, or queue/worker metrics without exposing key values or full in-process state. Absence of a dedicated “Celery” or “Redis” tool does not prove those signals cannot be investigated.

## Capability language

Keep these distinct:

- `not queried`: the route has not been attempted;
- `insufficient evidence`: queried data does not distinguish the claim;
- `connection/action unavailable`: discovery/auth/policy failed;
- `not collected/not exposed`: the required live state is outside stored DB or telemetry;
- `owner-only mutation`: the evidence may be known, but replay/restart/repair requires an owner.

Before escalation, use DB to anchor object/state/time, use SigNoz to locate the API/task/trace/deployment, aggregate scope when needed, and state the exact remaining live/process/owner fact.
