---
name: dify-support
description: 调查 Dify SaaS 工单、核验支持回复并起草客户答复；用于 Cloud 运行时、配置、计费、插件和产品问题，不用于 Enterprise 或私有部署支持。
---

<!-- Public export: deployment-specific identifiers or personal examples were replaced with placeholders on 2026-10-09. -->
# Dify SaaS Support

完成当前客户问题的调查及所需的未发送草稿。根据证据选择调查深度；持续寻找客户可执行的恢复办法。一个简短答复也需要足够的证据，但不需要运行无关调查路线。

## Intake and authorization

- 工单任务先通过 OpenConnector 的 `zendesk` 连接刷新最新 `conversation_log`，按需读取 ticket/comments 和相关附件；Messaging 图片用 `list_attachments` 定位。不要建立本地工单镜像。
- “处理工单”“帮我回”“提交给开发”默认交付分析和草稿。内部备注、状态、分配、标签、Linear/Feishu/Notion 写入、退款或 Admin 操作，必须有用户对具体动作的明确授权。公共回复在 OpenConnector 的 `zendesk` 连接中不可用，不得声称已发送。
- 需要查询 Zendesk 或执行明确授权的工单/Admin 操作时，读 [ticket access and actions](references/ticket-access-and-actions.md)，保留工具 dry-run、身份、确认和审计要求；已有具体执行授权不另设规划批准关卡。
- 先确认工单属于 Dify SaaS，并记录客户对象、入口、时间与时区，以及影响判断所需的运行时版本。不能因环境未知就默认 SaaS；环境、源码对齐或输出模式有疑问时读 [case context](references/case-context.md)。若确认是 Enterprise、私有部署、Helm 或 Docker Compose 场景，停止本 Skill 的调查并转用 `$dify-enterprise-support`；禁止用 SaaS 生产数据推断客户环境。

## Tool and external-service preflight

- 除刷新最新 Zendesk 线程所需的读取外，在开始其他调查调用前先读 [tool and service map](references/tool-and-service-map.md)，并在工作更新或内部分析开头列出本单的工具与外部服务预检。
- 区分四层：OpenConnector 控制工具、provider/命名连接、远端业务动作、OpenConnector 之外的外部服务或 owner-only 系统。连接存在不等于所需动作、权限、数据覆盖或时间范围可用。
- 对每项写明 `required`、`conditional`、`unavailable/owner-only` 或 `not needed`，以及用途、连接/动作族、所需标识或时间范围、读写风险和选择原因。先列计划，再调用非 Zendesk 工具；新证据改变路径时更新该清单。
- 通过实时 `list_connections`、`custom_mcp.list_tools`、`search_actions` 和 `get_action_guide` 核验将要使用的能力，不缓存工具数量或 schema。不要为了完成清单而调用与问题无关的服务。

## Evidence and continuity

- 每个重要结论标记 `confirmed`、`evidence-supported`、`ruled out` 或 `unknown`，并保留对象、时间、版本/路径及实际观测依据。历史工单、常见经验、当前快照、源码可能路径或空日志不能独自证明根因。
- 区分症状、首个失败边界、触发条件、根因、放大因素、恢复事件和当前状态。源码说明可能怎样运行；客户对应 DB、trace、日志或工件证明实际执行了什么。
- 客户补充信息时更新同一调查：新事实、已推翻假设、未答问题、已尝试操作、各次测试的实际发布配置。核验旧回复，纠正被新证据否定的结论，不因用户要求“通俗一点”而丢失未完成的恢复问题。
- SaaS 源码通过 OpenConnector 的 `dify-source` 连接查询，并核实 runtime/ref/commit 和客户路径。版本无法对齐时记录该缺口，继续其他独立证据，不能把本地源码、`main` 或相近版本当部署事实。

## Choose the needed route

| Case | Evidence and references |
| --- | --- |
| 稳定产品定义、政策或确定性配置问答 | 核实适用环境和当前权威资料，直接回答；无需完整事故矩阵、跨租户聚合或因果台账 |
| 单对象运行时故障、原因分析或回复正确性核验 | 读 [investigation method](references/investigation-method.md)，记录重要因果/状态证据和未知项；按失败路径选数据源 |
| 间歇故障、延迟、429/5xx、队列压力、恢复事件、回归或平台影响假设 | 上述调查方法，加 [SaaS deep investigation](references/saas-deep-investigation.md)；跨模块/症状变化时才查 [scenario matrix](references/saas-scenario-matrix.md) 的适用行。验证影响样本、基线和贡献集中度，范围不等于根因 |
| 需要 DB、SigNoz、Cloudflare、Source、GitHub 或内部知识证据 | 先按 [tool and service map](references/tool-and-service-map.md) 标明作用和选择理由，再读 [evidence sources](references/evidence-sources.md) 的对应部分；不逐个调用所有后端 |
| 客户被阻塞或提出修复/workaround | 读 [recovery validation](references/recovery-validation.md)，核查前提、操作、预期、验证级别、代价及恢复原配置后的行为 |
| 已知缺陷、历史工程讨论、修复进度或升级建议 | 读 [engineering evidence](references/engineering-evidence.md)；issue、PR 合并、部署、客户恢复分别核实 |
| 计费/额度/预留/Stripe 时间线 | 读 [billing quota runtime](references/billing-quota-runtime.md)；退款再读 [refunds](references/refunds.md) |
| 客户草稿、Agent 回复评审、开发/Linear 草稿 | 读 [reply and handoff](references/reply-and-handoff.md) 的适用部分，以及 [reply templates](references/reply-templates.md) |
| 实时首轮后仍有重要诊断缺口，或需要沉淀已确认的排障经验 | 读 [diagnostic learning](references/diagnostic-learning.md)；通过 OpenConnector `notion` / `default` 使用 `Dify SaaS 排障经验库`，不再依赖独立 learning Skill 或 MCP |
| 明确要求整理正式 SaaS 知识，或本次已有确认的可复用知识增量 | 读 [OpenConnector routing](references/openconnector-routing.md)，经 OpenConnector `feishu` / `default` 在明确授权后写入飞书 `SaaS知识库`；普通工单不触发例行知识审计或知识库写入 |

## Data and tool boundaries

- 涉及 Zendesk、Dify 官方文档/官网、Dify DB、SigNoz、Source、GitHub、Linear、飞书、Notion 或 Cloudflare 时，先读 [OpenConnector routing](references/openconnector-routing.md)，通过对应 OpenConnector provider 和命名连接动态发现并调用工具；不得绕过 OpenConnector 直连 MCP、CLI 或网页。
- 使用 DB/SigNoz 判断任务、队列或恢复状态，或准备以工具能力不足为由交接时，读 [DB and SigNoz capabilities](references/db-signoz-capabilities.md)。区分未查询、证据不足、连接不可用和接口不覆盖；先核实实际遥测再声明无法查询。
- 业务数据库永久只读，通过 OpenConnector 的 `dify-db` 连接调用 `db_gateway_query`；不执行 DML/DDL 或其他修改 SQL。schema 不确定时查 `information_schema`。修复走另有授权和审计的 Admin/SRE/工程通道。
- SigNoz 通过 OpenConnector 的 `dify-signoz` 连接查询，优先适用的结构化工具；高级只读查询才用 `sz_run`。连接或工具不可用时按 OpenConnector 路由规则记录缺口，不擅自切直连 MCP、本地 CLI 或 UI。
- TiDB 向量存储按 `prod.tidb_auth_bindings.cluster_name` 映射；默认交付可供 TiDB Cloud Web 搜索的 cluster_name，由用户检查 Web 数据。不要使用 `ticloud`，详见 evidence-sources 的 TiDB 部分。
- 飞书读取统一通过 OpenConnector `feishu` provider 的 `default` 连接（当前身份“示例用户”）。读取范围是该身份有权访问的全部知识库/Wiki 空间与节点、Drive 文档和文件，以及群聊、单聊和线程聊天记录；按工单锚点与时间窗口定向搜索，并完成相关分页，不做无目的全组织内容扫描。正式产品知识仍以飞书 `SaaS知识库`（`YOUR_SAAS_KNOWLEDGE_BASE_TOKEN`）为权威来源，其他文档和聊天作为需核实的辅助或工程证据。不要直连 Lark/Feishu 工具，不要用 Confluence 或本地旧知识替代。
- Notion 排障经验统一通过 OpenConnector `notion` provider 的 `default` 连接读取或在授权后写入。只使用 `Dify SaaS 排障经验库` 中已脱敏、边界清楚的条目；历史经验只能指导下一步核查，不能作为当前工单根因、恢复或版本状态的证据。具体查询、候选沉淀和发布边界见 [diagnostic learning](references/diagnostic-learning.md)。
- Zendesk Help Center 不作为官方文档、产品知识或工单证据来源；不要调用 `search_help_center` 或 `get_article`。官方公开说明统一通过 OpenConnector `custom_mcp` provider 的 `dify-docs` 连接读取：文档先用 `official_docs_search` 且 SaaS 工单固定选择 `source=official_docs_community`，再以 `official_docs_read` 读取精确页面；官网内容用 `official_site_list`、`official_site_search` 和 `official_site_read`。核实页面 URL、SaaS 路径、语言和刷新时间，长内容按 `next_offset` 继续读取，并在客户草稿中引用 `docs.dify.ai` 或 `dify.ai` 原始链接。连接或页面不可用时记录具体缺口，不直连网页、不使用 Enterprise 文档、Help Center 或第三方转载兜底。
- 客户草稿不暴露内部 SQL、日志、私有链接、凭据或无依据工程承诺。遵循用户指定语言和渠道；解释哪个工具/节点、哪里不一致、现在如何处理。
- 最终交付未发送的客户拟回复时，按 [reply templates](references/reply-templates.md) 提供英文和中文的独立纯文本块；用户明确指定其他语言或格式时从其要求。区分供客户发送的版本与供用户核对的译文。

## Customer reply verification gate

回复客户必须有十足证据或经过验证。任何客户草稿（含简短跟进）交付前都执行本门槛，细则见 [reply and handoff](references/reply-and-handoff.md) 的 Verification gate。

- 草稿中的每一句事实、原因、当前状态、配置/命令、版本兼容、第三方行为、政策/支持范围、建议和承诺，都必须是 `confirmed`：由本次调查实际读取的证据直接证明，包括客户对象的 DB/日志/trace/附件、对齐运行版本的源码、当前权威文档、第三方官方文档、负责人本期明确确认，或已实际执行的验证。
- `evidence-supported`、推断、历史工单、AI 内部备注、以往回复、记忆和常见经验都只是线索，必须先核实，否则从草稿删除。文档、历史回复或内部讨论里的说法同样要先核对机制和适用范围，不能照搬。
- 无法核实、但客户需要知道的点，只能写成真实的进展状态（例如"此点正在与工程确认"），并列为阻塞项；不得用看似合理的答案补位。修复或 workaround 只能按实际达到的验证级别描述。
- 交付时同时给出逐句核实清单和发送结论。只有全部陈述为 `confirmed` 且没有阻塞项时，才能标为"可发送"；否则标为"暂不可发送"，并列出未核实的陈述、缺少的证据和确认途径。

## Completion

答案应覆盖客户真实问题、证据边界和当前可执行步骤。故障调查还应给出已验证级别明确的恢复路径；若尚不能验证，则交付已检查候选、失败原因和最小剩余客户/owner 核查。完成会改变结论的可用独立检查后再交接；证据充分后不无限扩查。草稿或交接不等于事故已经解决。客户草稿未通过核实门槛时，不得称为最终版、可发送或可直接回复。

只对有独立收益的证据路线委派，服从全局子代理上限；实时写操作留在主代理，子代理不能扩大授权。
