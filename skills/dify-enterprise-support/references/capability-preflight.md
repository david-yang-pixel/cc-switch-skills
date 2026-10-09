# Capability preflight

Use this reference before investigating an Enterprise ticket. Dependency declarations and configured endpoints do not prove that tools are callable in the current session.

## Required checks

Inspect the live tool surface and select only the connectors needed for the current route:

- OpenConnector `custom_mcp` connection `zendesk`: required for a live ticket, customer-visible thread, attachments, requester, and history;
- Helm Watchdog: required for Enterprise release composition, component-image mapping, upgrade paths, and unskippable versions;
- OpenConnector `custom_mcp` connection `dify-source`: required for exact-tag or exact-commit implementation claims;
- OpenConnector `linear` connection `default`: required when claiming a known issue, owner diagnosis, fix, or released version;
- OpenConnector `custom_mcp` connection `github-mcp`: supplemental discovery when `dify-source` lacks a repository or exact ref; it does not replace release alignment;
- OpenConnector `custom_mcp` connection `dify-docs`: required for claims based on current public Dify documentation or `dify.ai` website content.

For every OpenConnector route, follow [OpenConnector routing](openconnector-routing.md): discover the named connection's current tools/actions and make one lightweight read-only call that proves the relevant operation works. Check Helm Watchdog separately when the ticket needs release composition or upgrade evidence. A dependency being configured, reachable, or listed is not sufficient when its tools are absent or authentication fails.

## Failure behavior

- If Zendesk is unavailable, do not claim that pasted or cached text is the latest ticket state. State that live ticket freshness is unverified.
- If Helm Watchdog is unavailable, do not give a definite Enterprise-to-component mapping, supported upgrade path, or unskippable-version conclusion from memory. Use exact release artifacts only if they independently prove the same relationship; otherwise mark it `unknown`.
- If OpenConnector connection `dify-source` cannot resolve the exact ref, do not substitute `main`, a SaaS runtime ref, or a neighboring version.
- If OpenConnector Linear is unavailable, do not bypass it with a direct Linear MCP or URL, and do not claim that a symptom is or is not a known issue, fixed, scheduled, or released. Source and release evidence may still support a narrower product-version conclusion.
- If `dify-docs` is unavailable, do not use Zendesk Help Center or third-party copies as a substitute. Versioned Enterprise Docs and exact source may still support a narrower version-scoped conclusion.
- If a required source is unavailable, continue only with conclusions supported by the remaining evidence. Record the unavailable capability and its effect in the internal evidence table.

Never invent a tool name or silently degrade a definite claim to an unsupported guess.
