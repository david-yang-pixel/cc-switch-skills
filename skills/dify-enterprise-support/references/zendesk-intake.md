# Zendesk intake

Use this reference when a ticket ID or Zendesk case is in scope.

## Current ticket first

Read current live Zendesk data through OpenConnector connection `zendesk` before using pasted text or history. Follow [OpenConnector routing](openconnector-routing.md). Prefer one complete read when supported; otherwise retrieve metadata and the customer-visible thread separately.

Collect:

- ticket ID, subject, form, status, priority, tags, created/updated timestamps;
- requester identity and organization;
- product, Enterprise version, environment, deployment method, language and region;
- latest customer question, all customer-visible replies, attachments and inline images;
- explicit object identifiers, domains, namespaces, release names, Pod names, request/run/trace IDs, error text and timestamps.

Known custom-field mappings may help interpret raw responses, but verify them against live field names because Zendesk schemas can change:

- Product: `40647962111252`
- Version: `40647984651156`
- Environment: `40648116815636`

Preserve the raw values. Normalize versions only in a separate comparison field.

## Channel routing

- If `from_messaging_channel=true`, use the Messaging conversation log as the primary customer thread.
- For web-form or email tickets, use public ticket comments as the primary thread.
- Treat internal notes, bot audit entries, routing events, and status automation as operational context, not customer statements.
- Avoid reading the same conversation twice unless the ticket `updated_at` changed or a forced refresh is needed.

Useful Zendesk reads include `get_ticket`, `get_ticket_conversation_log`, `get_ticket_comments`, `list_attachments`, `download_attachment`, `get_user`, and `search_tickets`. Use only tools returned by OpenConnector discovery for connection `zendesk`; if a named tool is absent, inspect the current connection tool surface rather than inventing a call. For complete conversation, comment, attachment, or ticket-search reads, resume from `nextCursor` until `hasMore=false`; a subrequest-limit interruption is resumable partial data, not a complete result.

## Attachments

List attachments early when screenshots, logs, configuration, Helm values, or diagnostic bundles are mentioned. Inspect the original artifact when possible. A screenshot without environment, time, and surrounding action proves only what is visibly shown.

Do not request or reproduce:

- passwords, private keys, API tokens, license files, complete Kubernetes Secrets;
- full unredacted environment files;
- unrestricted database dumps or broad production log archives.

Ask the customer to redact secrets and personal data. Prefer a narrow time window and the affected component.

## Missing-information gate

Before version-specific diagnosis, establish at least:

1. affected environment;
2. exact Enterprise/Chart version;
3. symptom or exact question;
4. affected object or request path;
5. occurrence time and timezone;
6. raw error, log, screenshot, or configuration evidence when the question is not purely documentary.

If the Environment field contains multiple values, do not assume all are affected. Ask which environment reproduced the issue.

## Write boundary

This Skill never sends a public reply or public comment. Public-reply wording from the user authorizes drafting only. Other Zendesk mutations require a separate explicit request, OpenConnector `confirmOperation="write"`, downstream dry-run/confirmation gates, and a tool that actually supports the mutation.

## Final freshness gate

Record the ticket `updated_at`, latest customer-visible event/message identifier, and attachment set from the initial read. Immediately before producing the final evidence table and customer draft:

1. refresh the ticket and the authoritative customer-visible thread;
2. compare `updated_at`, the latest customer message, public replies, and attachments with the initial snapshot;
3. if anything changed, incorporate the new evidence and re-evaluate the scope, environment, version, failure boundary, and proposed response;
4. do not present a draft based on the stale snapshot.

This is one final freshness check, not continuous polling. Never resend or mutate the ticket during the check.
