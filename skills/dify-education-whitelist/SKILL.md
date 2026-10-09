---
name: dify-education-whitelist
description: Use when judging whether a school, education email, email domain, university domain, education discount request, or 教育邮箱/教育域名/教育优惠白名单 request qualifies for Dify SaaS education whitelist handling.
---

<!-- Public export: deployment-specific identifiers or personal examples were replaced with placeholders on 2026-10-09. -->
# Dify Education Whitelist

## Overview

Judge whether an email domain can enter the Dify SaaS education discount whitelist. The whitelist is domain-based: evaluate the school domain behind the email, not the individual mailbox.

Use current evidence. Route Dify internal knowledge and official public Dify material through OpenConnector:

- Feishu uses provider `feishu`, connection `default`. Search the canonical `SaaS知识库` folder (token `YOUR_SAAS_KNOWLEDGE_BASE_TOKEN`) first; other knowledge bases, documents, and relevant chat history visible to the connected identity are supporting evidence only.
- Official Dify documentation and website content use `custom_mcp`, connection `dify-docs`, with `official_docs_search/read` or `official_site_list/search/read`. Zendesk Help Center is not an official source or fallback.
- Existing whitelist state may be checked through the read-only OpenConnector `dify-db` connection when its current schema and tools support the question. Never write the business database.

Before using a route, verify the named connection with `list_connections`; discover Feishu actions with `search_actions` and Custom MCP tools with `custom_mcp.list_tools`, then invoke only the discovered read operation. Do not bypass OpenConnector with direct Lark/Feishu tools, a direct MCP namespace, Confluence, or legacy endpoints. A visible connection does not prove access, coverage, or freshness.

The institution's own website and recognized public education registries remain valid external evidence for verifying that the institution exists and controls the requested domain. They do not replace Dify policy evidence.

## Intake

Extract and normalize:

1. `email_domain`: if given an email, take the part after `@`; if given a domain, use it directly.
2. `school_name`: use the provided name, or identify it from domain evidence.
3. `request_reason`: optional support context, not a pass condition.

Normalize domains to lowercase, trim whitespace, and remove one trailing dot. Reject values containing spaces, slash, backslash, path fragments, or no top-level domain. For Admin records, store only the domain.

## Decision Workflow

1. Reject consumer or generic company email domains.
   Examples: `gmail.com`, `qq.com`, `outlook.com`, `163.com`, `yahoo.com`, and similar personal mailbox providers. Do not whitelist ordinary company mail unless strong evidence shows it is an official higher-education domain.

2. Verify the institution.
   Use the institution's official website and recognized public education registries. Confirm it exists and is a higher-education institution such as a university, college, or equivalent post-secondary institution.

3. Verify domain ownership.
   Confirm the email domain belongs to the same institution. Standardize multilingual names and abbreviations to the official English full name when possible.

4. Reject unsupported institutions.
   Reject K12 schools, training institutions, non-education organizations, unverifiable institutions, and ambiguous names that cannot be uniquely mapped.

5. Choose the whitelist domain.
   Prefer the confirmed institution root domain. A subdomain matches only when it equals the whitelisted domain or ends with `.` + whitelisted domain. Do not let `evilharvard.edu` match `harvard.edu`.

6. Check existing support when possible.
   If the OpenConnector `dify-db` read-only route or an authorized Admin read shows the domain already exists and is active, treat it as already supported. If it exists but inactive, treat it as a reactivation request requiring explicit approval. Record `unknown` when neither route can prove current state.

## Outputs

Use one of these decisions:

- `PASS`: eligible for whitelist.
- `REJECT`: not eligible; include the reason.
- `NEED_MORE_INFO`: missing school/domain or evidence is ambiguous.
- `ALREADY_SUPPORTED`: domain is already active in the education whitelist.

For `PASS`, output the Admin-ready record:

```json
{
  "name": "Official English Institution Name",
  "domain": "example.edu"
}
```

If a school has multiple independent education email domains, output one record per domain.

## Admin Handling

Education whitelist changes are high-risk Admin operations. Default to analysis and recommendation only; do not apply live changes unless the user explicitly asks. Read authorization does not authorize insertion, update, ticket closure, or any other mutation. Never use `dify-db` for a write.

Admin facts should include requested `domain`, `name`, optional `source`, desired `is_active`, and current whitelist state if available. Applying should insert/update the domain and close the Admin ticket only after confirmation.

## Customer-Safe Reply Notes

Do not expose Admin platform internals, database tables, internal knowledge links, or implementation details to customers. Say only whether the domain can be supported, whether more information is needed, or why it cannot be supported.

Common rejection wording:

- School cannot be verified as a higher-education institution.
- The domain does not appear to belong to the school.
- K12 schools and training organizations are not currently supported.
- Personal or generic email providers cannot be added to the education whitelist.

## Examples

| Input | Decision | Notes |
| --- | --- | --- |
| `student@cs.harvard.edu` + Harvard University | `PASS` | Output `Harvard University` / `harvard.edu` if official evidence confirms parent domain. |
| `user@gmail.com` + any school name | `REJECT` | Personal mailbox provider. |
| `student@training-example.com` | `REJECT` | Training institution, not higher education. |
| `student@evilharvard.edu` | `REJECT` | Domain boundary mismatch; does not match `harvard.edu`. |
| `University name only` | `NEED_MORE_INFO` or `PASS` | Pass only if the official education email domain is clear. |

## Common Mistakes

- Do not accept a domain only because it ends in `.edu`; still verify institution and ownership.
- Do not add top-level domains such as `edu`, `edu.cn`, or `ac.uk`.
- Do not add a full email address as the domain.
- Do not treat alumni/community domains as eligible unless they are official school education email domains and policy evidence is clear.
- Do not manually add ambiguous domains; ask for the official school name or official email-domain evidence.
