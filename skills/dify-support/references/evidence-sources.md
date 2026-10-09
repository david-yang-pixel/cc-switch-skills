# Evidence Sources

Choose sources from the actual execution chain, not convenience. Follow [OpenConnector routing](openconnector-routing.md) for every covered provider.

1. Refresh the latest Zendesk Messaging thread and relevant attachments.
2. Establish customer object, path, exact UTC window, and identifiers.
3. For product behavior, limits, policy, or SOP, search canonical Feishu `SaaS知识库` first; use other accessible Feishu documents/chat as supporting evidence and verify freshness/authority.
4. For customer-facing official guidance, query OpenConnector `dify-docs`: `official_docs_search/read` for `docs.dify.ai`, and `official_site_list/search/read` for `dify.ai`. Never use Zendesk Help Center or third-party copies as official information.
5. Query `dify-db` when the answer depends on current workspace/app/run/dataset/plugin/billing state. Keep it read-only and inspect `information_schema` before schema-sensitive SQL.
6. Query `dify-signoz` for runtime errors, traces, queues, workers, APIs, slowness, stuck work, reserves, webhooks/cron, dependencies, or capacity evidence. Discover fields/services first and use `sz_run` only when structured tools cannot express the read.
7. Query Cloudflare only when the path implicates edge/CDN/WAF/DNS/TLS, HTTP analytics, or Workers. Correlate edge evidence with SigNoz origin evidence.
8. Align source only after the customer path and runtime/plugin version are known. Source describes possible behavior; live evidence proves the executed branch.
9. When a defect or fix is plausible, read [engineering evidence](engineering-evidence.md), Linear, original Feishu discussion, and relevant GitHub PR/release evidence. Distinguish proposal, issue status, merge, release, deployment, and customer recovery.
10. Only after an independent live first pass leaves a material gap, query published sanitized Notion lessons per [diagnostic learning](diagnostic-learning.md). Historical experience guides the next check and never proves the current ticket.

## Database and TiDB

- `prod`: SaaS application/workspace/run/dataset/account state.
- `billing`: subscription, Stripe mirror, features, quotas, promotions.
- `plugin`: plugin installation/runtime records.
- Use `prod.tidb_auth_bindings.cluster_name` only to map a workspace to the TiDB Cloud resource. TiDB Cloud Web is the owner-visible source for vector storage; do not use `ticloud`.
- Database mutation is prohibited. Route repairs through an approved Admin/SRE/engineering path.

## SigNoz

Prefer service/field discovery → aggregate scope → matching logs/spans → trace details → capacity metrics. An empty result is not negative evidence until field names, deployments, UTC conversion, retention, and signal freshness are checked. See [DB and SigNoz capabilities](db-signoz-capabilities.md) and [SigNoz MCP](signoz-mcp.md).

## Provider and plugin evidence

For plugin/model-provider cases, read [SaaS plugin troubleshooting](plugin-troubleshooting.md); verify workspace installed version, Marketplace latest version/package, aligned installed-version source, and the provider/API boundary. Do not generalize one installed version into a product-wide capability claim.

## Unavailable hops

Mark each missing DB, SigNoz, Source, Feishu, Notion, Linear, GitHub, Cloudflare, provider, browser, billing, storage, or owner path independently. Continue other useful routes; do not hide the gap under “backend unavailable” or bypass OpenConnector.
