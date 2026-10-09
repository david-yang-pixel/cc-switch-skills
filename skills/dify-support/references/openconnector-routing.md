# OpenConnector Routing

Use OpenConnector as the only route for the ten support evidence services listed in [tool and service map](tool-and-service-map.md). Do not bypass it with direct MCP namespaces, public web search/open, provider CLIs, dashboards, or legacy HTTP endpoints.

## Discover live capability

1. Call `list_connections` and verify the exact provider and connection name.
2. For Custom MCP connections, call `custom_mcp.list_tools`; treat its current tool name, schema, and `operationType` as authoritative.
3. For Linear, Feishu, Notion, and Cloudflare, call `search_actions` for the provider. Read `get_action_guide` when the schema or behavior is not already clear.
4. Invoke only the discovered action through `custom_mcp.execute` or `execute_action` with the matching connection.
5. Inspect pagination/continuation metadata after every list or search page.

Custom MCP pagination is exposed at `structuredContent["io.oomol.open-connector/pagination"]`. For `official_docs_read` and `official_site_read`, follow `data.next_offset` until null when the conclusion depends on later content.

## Connection registry

| Capability | Provider | Connection |
| --- | --- | --- |
| Zendesk | `custom_mcp` | `zendesk` |
| Dify business DB | `custom_mcp` | `dify-db` |
| SigNoz | `custom_mcp` | `dify-signoz` |
| Source | `custom_mcp` | `dify-source` |
| GitHub | `custom_mcp` | `github-mcp` |
| Official docs and Dify website | `custom_mcp` | `dify-docs` |
| Linear | `linear` | `default` |
| Feishu | `feishu` | `default` |
| Notion | `notion` | `default` |
| Cloudflare | `cloudflare_mcp` | `default` |

## Read and write boundary

- Zendesk, DB, SigNoz, Source, GitHub, Linear, Feishu, Notion, docs, and Cloudflare investigation is read-only unless the user explicitly authorizes the exact external write.
- Business DB stays read-only even when a repair is requested. Route repair through an approved Admin/SRE/engineering workflow.
- Linear issue/comment/project changes, Feishu sends/document changes, Notion candidate/status/content changes, and GitHub issue/PR/comment/review changes require action-specific authorization.
- `cloudflare_mcp.execute` is a generic write-capable runner. For support evidence, call it only after `cloudflare_mcp.search`, only with the `default` read-only connection, and only for a bounded GET/list/read operation. Never use it for POST, PUT, PATCH, DELETE, deploy, purge, or configuration changes without a separate authorized mutation workflow.
- A previous import, database creation, connection setup, or read authorization does not authorize future writes.

## Provider-specific rules

- Feishu may read the full corpus visible to the connected identity, but searches must be anchored to the ticket and relevant time window; full scope does not authorize unrelated bulk collection.
- Notion reads only the configured `Dify SaaS 排障经验库`; follow [diagnostic learning](diagnostic-learning.md).
- Dify docs uses `official_docs_search/read` and `official_site_list/search/read`. Zendesk Help Center remains excluded even when actions are discoverable.
- Cloudflare account and zone must be resolved through read operations. Keep the company account separate from personal accounts.

If discovery, connection, authentication, permission, retention, or the required action is unavailable, record the exact failed hop and its effect on the conclusion. Continue independent evidence routes without bypassing OpenConnector.
