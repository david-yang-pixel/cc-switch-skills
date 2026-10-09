# Linear known issues and fixes

Use this reference when a customer symptom may match an existing defect, regression, limitation, planned change, or released fix.

## Connector and permissions

Use OpenConnector provider `linear`, connection `default`. Verify it with `list_connections`, discover the current read actions with `search_actions`, and inspect `get_action_guide` when the schema is unclear. Do not use a direct Linear MCP, public Linear URL, web search, copied ticket text, or memory. Investigation is read-only by default. Do not create, update, comment on, relate, or change a Linear issue unless the user separately and explicitly requests that mutation.

## Search sequence

Start from current-ticket evidence and search in this order:

1. exact error code or distinctive error text;
2. API path, configuration name, migration command, image, or component;
3. feature name plus symptom;
4. affected Enterprise/core version plus component;
5. exact Linear identifier when the ticket or release evidence provides one.

Use the discovered OpenConnector Linear search action for discovery. For each plausible result, use the currently discovered issue, comment, relation, and release actions rather than assuming fixed action names:

1. read the full issue with relations, releases, and customer needs when supported and relevant;
2. read all applicable comment pages to inspect owner findings, corrections, current status, workaround, and release discussion;
3. follow duplicate, blocked-by, blocks, or related issue links when they materially change the diagnosis;
4. read the associated release or release-note evidence when a fix version is claimed;
5. align the claimed affected/fixed version with Helm Watchdog and exact source refs.

Search results are leads, not evidence. Similar wording does not establish that the customer's environment hit the same defect.

## Evidence rules

Classify Linear findings as:

- `hypothesis`: search result or issue summary resembles the symptom but the environment, version, or failure boundary is not aligned;
- `evidence-supported`: the full issue or owner comments describe the same boundary and an applicable version, but customer-side confirmation or released-source alignment is incomplete;
- `confirmed`: current customer evidence matches the issue boundary and affected version, and the corresponding source/release evidence aligns;
- `ruled out`: the issue affects a different component, path, version, configuration, or failure mode.

Do not infer any of the following:

- `Completed`, `Resolved`, or `Canceled` means the customer's installed release contains the fix;
- a merged PR means an Enterprise image was built or shipped;
- a release association means the customer completed that upgrade or migration;
- no search result means no known issue exists;
- a workaround or recovered customer object means the underlying defect is fixed.

For a fix claim, establish this chain:

```text
matching customer symptom and failure boundary
-> Linear owner diagnosis
-> fix PR or immutable commit
-> Enterprise release/component mapping
-> customer's installed version relationship
-> customer-path verification when claiming recovery
```

If any link is missing, state the narrower supported conclusion and keep the rest `unknown`.

## Internal record

Keep a concise internal record:

```text
Linear issue:
Status and last meaningful update:
Customer symptom match:
Affected version evidence:
Fix commit or PR evidence:
Released Enterprise version evidence:
Customer version relationship:
Workaround:
Conclusion status:
Remaining gap:
```

## Customer-safe output

Do not copy internal Linear titles, comments, customer names, private attachments, roadmap dates, unreleased details, or internal URLs into the customer draft. Translate only the verified customer-safe conclusion, for example:

```text
我们确认该现象与一个已知问题的失败边界一致。该问题已在 Dify 企业版 <版本> 中修复；您当前使用的 <版本> 尚未包含该修复。
```

If release inclusion is not confirmed, say that the issue is under investigation and avoid promising a version or delivery date.
