# OpenConnector Cloudflare Read-Only Evidence Route

Use only when Dify SaaS evidence implicates edge/CDN/WAF/DNS/TLS, HTTP analytics, or Workers observability. This is one hop in the execution chain, not a replacement for Zendesk, DB, SigNoz, source, or owner evidence.

## Boundary

- Use OpenConnector service `cloudflare_mcp`, connection `default`, for the `LangGenius, Inc.` company account. Do not expose or query a personal account.
- Discover the live action surface with `search_actions`; inspect `get_action_guide` when needed.
- Use `cloudflare_mcp.search` to locate the relevant Cloudflare API read operation, then `cloudflare_mcp.execute` only for the bounded GET/list/read operation found by search.
- Reject POST, PUT, PATCH, DELETE, purge, deploy, configuration, membership, token, DNS/WAF/TLS mutation, and any other state change without a separate explicitly authorized workflow.
- Never use direct Cloudflare MCP, Wrangler, Dashboard, `cloudflare-support`, or another alias as fallback.

## Minimum anchors

- hostname and zone;
- exact UTC start/end time;
- path, method, status, and error text;
- region/colo when available;
- CF-Ray, request ID, trace ID, app/run ID, and source IP only when legitimately available and necessary.

## Procedure

1. Confirm the symptom implicates the edge; do not query Cloudflare for an internal workflow, queue, provider, or application error by default.
2. Resolve the company account and exact zone/hostname through reads.
3. Query only the applicable surface: HTTP analytics, security/WAF events, DNS, TLS/certificates, or Workers observability.
4. Correlate with the matching SigNoz origin trace/log window to determine whether the request reached origin and where status/latency arose.
5. Record query scope, observed result, permission/plan/retention/sampling limits, and causal role.

An edge-visible 4xx/5xx is not proof of WAF, rate limiting, origin failure, or root cause. Match a security event to hostname, path, action, rule, request anchor, and time before attribution. Current DNS/TLS state is not historical evidence unless reported-time state is available. Empty results are not negative evidence until account/zone, UTC, filters, dataset, permission, plan, retention, sampling, and freshness are checked.

Keep account IDs, raw security events, private hostnames, and internal rule names out of customer drafts.
