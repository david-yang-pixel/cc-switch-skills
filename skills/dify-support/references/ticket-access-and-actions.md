# Ticket Access And Actions

Use only OpenConnector `custom_mcp` connection `zendesk` for Zendesk access.

## Read workflow

1. Discover the current tools and operation types through `custom_mcp.list_tools`.
2. Read `get_ticket` and the latest `get_ticket_conversation_log`. For SaaS Messaging tickets, the conversation log is the primary customer thread; comments are secondary unless the request concerns comments.
3. Extract requester, tags, status, priority, timestamps, exact customer question, and all available identifiers.
4. Call `list_attachments` early when screenshots/files matter; Messaging images can be discoverable even when a ticket event has no ordinary attachment.
5. Read comments, users, views, or related tickets only when needed. Follow pagination until complete or report the continuation boundary.
6. Never call `search_help_center` or `get_article`; Zendesk Help Center is not an official-document or supporting-information source for this workflow.

## Mutation boundary

- “处理工单”, analysis, drafting, review, follow-up wording, or “帮我回” authorizes no Zendesk write.
- Public reply/comment is unavailable through the configured connection. Never claim a public response was sent.
- Add an internal note only when the user explicitly asks for an internal note. Pass the request wording through the tool's explicit authorization field and use its dry-run/confirmation controls.
- Change status only when the user explicitly names the desired status. Re-read `updated_at`, use optimistic-concurrency fields, and read back the final ticket. Solving requires all tool confirmations.
- Change priority, type, assignee, group, tags, or allowed custom fields only when explicitly requested. Do not smuggle a comment/body/attachment into metadata updates.
- Refunds, account changes, deletions, whitelists, and other Admin actions require verified requester authority, target identity, business conditions, exact authorization, confirmation, and audit evidence.

Status does not block an explicit inspection: read pending/hold/solved tickets when the user asks for follow-up, audit, or closure validation.
