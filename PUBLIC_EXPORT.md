# Public export notes

This repository contains a sanitized snapshot of 69 installed skills. The disabled `wechat-miniprogram` skill is not included.

## Deployment placeholders

Replace these values locally before using the corresponding integrations; do not commit real values:

- `https://openconnector.example.com/mcp`: your OpenConnector MCP endpoint.
- `https://helm-watchdog.example.com/api/v1/mcp`: your authorized release metadata MCP endpoint.
- `https://enterprise-releases.example.com/`: your authorized private release portal.
- `YOUR_SAAS_KNOWLEDGE_BASE_TOKEN`: your knowledge-base identifier.
- `YOUR_NOTION_DATABASE_ID` and `YOUR_NOTION_DATA_SOURCE_ID`: your diagnostic-learning store.

Personal names were replaced with sample names. A machine-specific sample migration path was made relative. Opaque sample document and pagination tokens were replaced with explicit placeholders. Missing name/description metadata was added to `verification-loop/SKILL.md` so clients can discover it. Python bytecode caches are excluded. All other original skill source content and supporting files are retained. No CC Switch database, provider credentials, account settings, logs, or local configuration files are included.

## Modified files

- `skills/dify-education-whitelist/SKILL.md`
- `skills/dify-education-whitelist/agents/openai.yaml`
- `skills/dify-enterprise-support/agents/openai.yaml`
- `skills/dify-enterprise-support/references/authoritative-sources.md`
- `skills/dify-enterprise-support/references/source-and-version-routing.md`
- `skills/dify-support/SKILL.md`
- `skills/dify-support/references/diagnostic-learning.md`
- `skills/dify-support/references/tool-and-service-map.md`
- `skills/lark-apps/references/lark-apps-db-execute.md`
- `skills/thesis-writer/SKILL.md`
- `skills/lark-approval/references/lark-approval-instance-form-control-parameters.md`
- `skills/lark-im/references/lark-im-reactions.md`
- `skills/verification-loop/SKILL.md`
