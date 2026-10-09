# OpenConnector routing

Use OpenConnector as the only route for these support evidence services:

| Capability | Provider | Connection |
| --- | --- | --- |
| Zendesk | `custom_mcp` | `zendesk` |
| Source repositories | `custom_mcp` | `dify-source` |
| GitHub | `custom_mcp` | `github-mcp` |
| Public Dify docs and website | `custom_mcp` | `dify-docs` |
| Linear | `linear` | `default` |

Before first use, call `list_connections` and verify the exact provider and connection. For Custom MCP connections, call `custom_mcp.list_tools` and treat the returned tool name, schema, and `operationType` as authoritative, then invoke the discovered operation through `custom_mcp.execute`. For Linear, call `search_actions`, read `get_action_guide` when needed, and invoke the discovered action through `execute_action` with connection `default`. Do not call legacy direct MCP namespaces, provider CLIs, public Linear URLs, or legacy HTTP endpoints for these services.

For paginated reads, inspect `structuredContent["io.oomol.open-connector/pagination"]`. Resume from `nextCursor` while `hasMore` is true. Treat `interruptionReason="subrequest_limit"` as a resumable partial result, preserve unique records already returned, and do not claim completeness until `hasMore` is false.

Any non-read operation requires explicit user authorization for that exact mutation and the matching OpenConnector `confirmOperation`. Downstream tool-specific safety fields still apply. If OpenConnector, discovery, authentication, the named connection, or the required action is unavailable, record the exact gap and continue only with conclusions supported by other evidence; do not bypass OpenConnector with a direct endpoint or local tool.

The versioned Enterprise Docs site, Enterprise release portal, public Helm release site, and Helm Watchdog remain separate Enterprise-specific sources until an equivalent named OpenConnector connection is available. Their availability does not authorize a mutation. Zendesk Help Center is excluded even if its actions appear during Zendesk discovery.
