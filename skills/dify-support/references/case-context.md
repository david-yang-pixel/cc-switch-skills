# Case Context

Read when environment, source alignment, or output intent is uncertain.

## Scope routing

- This skill handles Dify Cloud/SaaS only.
- Confirm Cloud/SaaS from the ticket, `cloud.dify.ai` path, or a workspace/account verified in SaaS production systems.
- If the case is Enterprise, private deployment, Helm, Docker Compose, or a customer-managed cluster, stop this skill and use `$dify-enterprise-support`.
- Do not query SaaS production data for an Enterprise customer and do not use Enterprise configuration assumptions for SaaS.
- If the answer is environment-independent, state that boundary rather than guessing the environment.

## Source alignment

For source-backed claims, identify the actual service/deployment and runtime image/ref first. Through OpenConnector `dify-source`:

1. use live runtime mapping to resolve the repository and deployed ref/commit;
2. search/read/log code only at that aligned ref;
3. keep plugin/model-provider claims scoped to the installed plugin version and relevant runtime;
4. if mapping fails, label exact deployed-source alignment `unknown` and continue independent DB, trace, log, configuration, artifact, and owner checks.

Do not substitute local source, GitHub `main`, a nearby release, an image name without immutable provenance, or docs as proof of the deployed code. Source explains possible behavior; customer-specific DB/trace/log evidence proves which path ran.

## Output intent

| User intent | Default output |
| --- | --- |
| “处理工单”, “看下这单”, pasted customer message | Analyze and prepare an unsent customer-safe draft |
| “怎么回复”, “帮我回” | Draft only; do not send |
| “提交给开发”, “转 Linear” | Chinese engineering/Linear draft; do not create unless explicitly authorized |
| “记一下”, “沉淀一下” | Prepare a candidate; no Feishu/Notion write without explicit authorization |
| Explicit knowledge-base or Notion write | Perform only the named write and verify it by readback |

When intent is unclear, choose read-only analysis plus an unsent draft.
