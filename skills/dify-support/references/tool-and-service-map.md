<!-- Public export: deployment-specific identifiers or personal examples were replaced with placeholders on 2026-10-09. -->

# Tool And External-Service Map

Read this before non-Zendesk investigation calls. List only services relevant to the ticket and classify each as `required`, `conditional`, `unavailable/owner-only`, or `not needed`.

For every selected route record: purpose, OpenConnector provider and connection, action family, identifiers/time window, read/write risk, and why it is needed. A visible provider or saved connection does not prove action availability, authorization, data coverage, or freshness.

## OpenConnector control plane

| Control tool | Role |
| --- | --- |
| `list_connections` | Prove the required named connection exists and is connected |
| `custom_mcp.list_tools` | Discover the current remote Custom MCP tools and `operationType` |
| `search_actions` | Discover native-provider actions for Linear, Feishu, Notion, and Cloudflare |
| `get_action_guide` | Read the live input schema and behavior of a selected action |
| `custom_mcp.execute` / `execute_action` | Invoke the discovered business operation through its named connection |

## Ten support evidence services

| Capability | Provider | Connection | Primary role |
| --- | --- | --- | --- |
| Zendesk ticket facts | `custom_mcp` | `zendesk` | Latest Messaging thread, ticket fields, comments, users, attachments; authorized internal/status/metadata writes only |
| Dify business DB | `custom_mcp` | `dify-db` | Read-only `prod`, `billing`, and `plugin` state |
| SigNoz | `custom_mcp` | `dify-signoz` | Logs, traces, metrics, scope aggregation, capacity evidence |
| Dify source | `custom_mcp` | `dify-source` | Runtime ref/commit alignment and code-path inspection |
| GitHub | `custom_mcp` | `github-mcp` | Public issue, PR, release, commit, and code evidence; read-only by default |
| Dify official docs and site | `custom_mcp` | `dify-docs` | `docs.dify.ai` content and `dify.ai` website pages; excludes Zendesk Help Center |
| Linear | `linear` | `default` | Engineering issues, comments, relations, projects, status, and fix progress |
| Feishu | `feishu` | `default` | All knowledge bases/Wiki nodes, Drive documents/files, and chat history visible to identity `示例用户` |
| Notion | `notion` | `default` | Sanitized diagnostic learning in `Dify SaaS 排障经验库` |
| Cloudflare | `cloudflare_mcp` | `default` | Edge/CDN/WAF/DNS/TLS, HTTP analytics, Workers, and API evidence |

Do not call all ten services by default. Select the smallest evidence set that can distinguish the material hypotheses.

## External or owner-only systems

These are not automatically callable through OpenConnector. Mark them independently when relevant:

- customer browser, WebApp/API path, screenshots, HAR, request/trace identifiers;
- model provider, third-party plugin API, Marketplace package/version;
- Stripe/payment processor or billing-owner state not exposed by the read-only DB;
- AWS/Kubernetes/Redis/Celery/Kafka state not present in captured telemetry;
- TiDB Cloud Web vector-storage facts after mapping `cluster_name` in the business DB;
- SRE/engineering owner actions such as replay, restart, refund, data repair, deployment, cache purge, or production mutation.

Do not bundle these as “backend checks.” State the exact unavailable hop and evidence needed.

## Preflight format

```text
工具与外部服务预检：
- <status> — <service/provider/connection>: <purpose>; <action family>; <identifiers/window>; <read/write boundary>; <why selected>
```
