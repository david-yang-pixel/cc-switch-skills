<!-- Public export: deployment-specific identifiers or personal examples were replaced with placeholders on 2026-10-09. -->

# Diagnostic Learning In Notion

This is the diagnostic-experience component of the main `dify-support` skill. Do not invoke a separate `dify-support-learning` skill or MCP.

## Store And Route

- OpenConnector provider: `notion`
- Connection: `default` (configure your own diagnostic-learning workspace)
- Database: `Dify SaaS 排障经验库`
- Database ID: `YOUR_NOTION_DATABASE_ID`
- Data source ID: `YOUR_NOTION_DATA_SOURCE_ID`

Use live OpenConnector discovery before the first Notion call in a task. Treat the configured connection, database identity, action schema, operation type, and policy returned at runtime as authoritative. If the named database cannot be retrieved, record the exact gap and do not substitute another Notion database, local Obsidian vault, SQLite store, separate learning MCP, or separate learning skill.

## Investigation-Time Read

Search historical experience only after the independent live pass has established the current ticket's environment, object, time window, execution chain, observed failure boundary, and material evidence gap.

1. Query the data source through `notion.query_data_source`. Default to `状态=已发布` and `隐私状态=已脱敏`; do not use `候选`, `已驳回`, or `已过期` as live investigation guidance.
2. Match on a focused component, symptom, failure boundary, evidence gap, or exact source reference. Query the smallest useful set and follow pagination.
3. Read the selected page body with `notion.retrieve_page_markdown`; a title or search snippet is not enough.
4. Use the lesson only to choose the next DB, SigNoz, Source, Feishu, Linear, Cloudflare, provider, browser, or customer-artifact check.
5. Re-prove every ticket-specific claim with current evidence. Record when a lesson is inapplicable because the environment, version, path, or symptom differs.

Notion is not the authority for formal product behavior, current policy, public instructions, or engineering status. Those remain with Dify official docs/site, canonical Feishu SaaS knowledge, live runtime evidence, and Linear as defined by the main skill.

## Candidate Capture

Prepare a candidate only when the investigation reveals a reusable diagnostic improvement, such as a missed evidence hop, recurring unsupported inference, decisive query order, failure boundary, or failed troubleshooting approach. Skip routine known cases, wording-only changes, customer-specific configuration, and unsupported root-cause guesses.

External Notion writes require explicit user authorization for the requested capture, import, update, publication, rejection, or expiration action. Authorization to investigate a ticket is not authorization to write a lesson.

When authorized, create or update the entry through OpenConnector `notion` / `default` and verify it by querying the data source and reading the returned page. A new or materially merged lesson starts as `候选` unless current evidence also satisfies the publication rules below.

Use the existing properties:

- `标题`
- `状态`: `候选`, `已发布`, `已驳回`, or `已过期`
- `场景分类`
- `症状关键词`
- `证据等级`: `已确认`, `证据支持`, or `未知`
- `适用版本`
- `适用边界`
- `来源编号`
- `隐私状态`: `待审核`, `已脱敏`, or `需清理`
- `最后验证时间`

Write the page body in Chinese. Translate selected non-Chinese source material into natural Chinese while preserving necessary product and API terms. Use this structure when it adds value:

- 核心判断
- 排查轨迹
- 判断依据
- 常见误区或失败复盘
- 验收标准

## Privacy And Evidence

Store only generalized, auditable reasoning traces. Never store raw chain-of-thought, customer messages or transcripts, attachments, emails, personal names, tenant/workspace/app/run identifiers, secrets, credentials, raw logs, high-entropy tokens, or private URLs. Minimal source references may use a safe ticket or Linear identifier when traceability is necessary.

Set `隐私状态=已脱敏` only after reviewing every property and the full page body. Otherwise use `待审核` or `需清理`.

Set evidence truthfully:

- `已确认`: the reusable pattern is supported by direct runtime, data, source-aligned, owner, or equivalent decisive evidence.
- `证据支持`: evidence favors the pattern but a material causal or coverage gap remains.
- `未知`: retained only as a review candidate; never publish or use as live guidance.

## Publication And Freshness

Mark a lesson `已发布` only when it is generalized, fully sanitized, has an explicit applicability boundary and anti-pattern, has `证据等级=已确认`, and its version/freshness claim is supported by current evidence. Version-sensitive pricing, entitlement, deployment, UI, provider, or policy facts must be revalidated before publication and before later use.

Promotion, rejection, expiration, and substantive edits are Notion writes and require explicit authorization. A published lesson still guides investigation only; it never replaces current ticket evidence.
