# Chinese reply draft and engineering escalation

Use this reference when producing the final customer draft or an internal engineering handoff.

## Mandatory customer-draft contract

- Output exactly one Chinese draft in a fenced `text` block unless the user explicitly asks for another format or language.
- If the user asks only for reply wording, return the draft without an internal assessment, but still run the evidence-grounded output gate and return the send verdict with any blockers.
- The draft is always unsent. Never call a public reply/comment tool, never paste it into Zendesk, and never claim it was delivered.
- Match the customer's actual question and technical level. Keep the first paragraph useful without requiring the customer to interpret internal investigation details.
- Include only statements that pass the evidence-grounded output gate below. If root cause is unknown, say so plainly and ask for the evidence required to locate it.
- Do not expose internal URLs, private repository names or links, SQL, Source MCP, Watchdog internals, internal tickets/PRs, raw support identifiers, credentials, or confidential release content.
- Do not promise a fix, release date, compatibility, successful rollback, or resolved state without matching evidence.

## Diagnostic evidence requests

- First decide whether the cause or first failing boundary can be located from the current ticket and version-aligned evidence.
- If it cannot be located, ask for the evidence set needed to identify the failing component and distinguish the remaining plausible causes; do not guess the cause or stop merely because the first evidence set is incomplete.
- The number of requested items is determined by the diagnosis, not by an arbitrary limit. Ask for all information required to make the next localization step conclusive, while excluding anything that cannot change the conclusion.
- Every requested item must have an internal one-to-one purpose: record which unresolved question or hypothesis it can confirm or rule out.
- Scope requests precisely to the affected environment, version, component or path, object/request identifier, and relevant time window. Request the effective configuration or relevant log section rather than a broad dump when only a subset is diagnostic.
- Do not add generic requests for logs, `kubectl` output, resource metrics, complete `values.yaml`, topology diagrams, or screenshots merely because they may be useful later.
- Ask for actual resource metrics only when the customer asks about measured consumption or sizing, or when resource pressure is a supported hypothesis that blocks diagnosis. Architecture or lifecycle questions alone do not justify `kubectl top` or container-resource collection.
- Do not expand a direct answer into unsolicited optimization, upgrade, capacity-planning, or security advice. Include a recommendation only when it directly answers the question or prevents a concrete risk in the proposed action.
- If no additional evidence is needed, do not end with an open-ended invitation such as “请提供更多信息以便进一步分析”.

## Evidence-grounded output gate

Customers receive only statements backed by conclusive evidence or verification. Run this gate before returning any customer draft, including short replies, follow-ups, and revisions.

Split the draft into atomic statements: facts, causes, current state, configuration or commands, version and compatibility claims, third-party behavior, support scope, recommendations, and promises. Every statement must be `confirmed` by at least one source read in this investigation:

- the current ticket, current customer message, attachment, or customer-provided runtime evidence;
- exact-version Helm values, rendered manifests, or component source mapped to the customer's release;
- applicable versioned Enterprise documentation or public Dify documentation read through OpenConnector `dify-docs`;
- a fully checked Linear issue with release evidence applicable to the customer's version;
- official documentation from the third-party vendor (cloud provider, database, model provider) for third-party behavior;
- explicit current confirmation from the responsible owner for support policy, validation status, release timing, or commitments;
- a verification actually performed, at the level actually reached.

Inferences, `evidence-supported` claims, historical tickets, AI-generated notes, prior replies, internal comments, and memory are leads, not sources. Verify them or remove them.

Apply these gates:

- No confirmed source: remove the statement. If the customer needs to know the gap, write only a true status (for example, that the point is being confirmed with engineering) and list it as a blocker. Never fill the gap from memory, typical behavior, or a plausible answer.
- Statements reused from documentation, prior replies, or internal discussion: check their mechanism, version scope, and applicability first; documentation can be wrong or out of scope.
- Source code or rendering shows capability but it was not run in the customer's or an equivalent environment: state the product behavior, say that it has not been validated in their environment, and ask them to validate in a test environment first.
- A proposed command or configuration change must have evidence that it applies to the exact version and failure boundary, plus a stated expected result; verify it by rendering or source when possible. Otherwise present it only as a diagnostic check, not a solution.
- Distinguish confirmed resolution, workaround, mitigation, and hypothesis. Never describe an untested suggestion as a fix.
- If the evidence supports an answer but not a root cause, answer the supported part and state what remains unknown.
- Absolute or general wording (`only`, `all`, `never`, `not supported`, `fixed`, `will`) needs evidence covering its full scope; otherwise narrow the statement.
- After every edit, re-read the whole draft sentence by sentence; new wording must pass the same gate.

Return a send verdict with the draft:

```text
发送结论：可发送 | 暂不可发送
阻塞项：<未核实的陈述 → 缺少的证据 → 确认途径或负责人>
```

Mark `可发送` only when every statement is `confirmed` and no blocker remains. Otherwise do not call the draft final, ready, or sendable.

## Draft shapes

Confirmed guidance:

```text
您好，

根据您当前使用的 Dify 企业版 <版本> 和 <环境/部署方式>，<直接回答或确认结果>。

<必要的操作说明或注意事项>。

如果操作后仍有异常，请提供 <能够定位该异常的证据及范围>，我们会继续协助确认。
```

Evidence request:

```text
您好，

目前可以确认 <已确认事实>，但现有信息还不足以判断问题发生在 <未确认边界>。

请补充以下信息：
1. <受影响环境和准确版本>
2. <精确时间和时区>
3. <经过脱敏的关键日志/配置/截图>

请勿提供密码、Token、私钥、许可证文件或完整 Secret。收到这些信息后，我们可以继续定位首个失败组件。
```

Version or upgrade guidance:

```text
您好，

您当前使用的是 Dify 企业版 <版本>。根据该版本的组件和升级要求，<版本范围内的确认结论>。

建议先 <备份/前置检查/升级路径中的下一步>，并在测试环境完成验证后再应用到生产环境。<必要的回滚或迁移边界>。
```

Adapt wording; do not mechanically include every placeholder or section.

## Internal assessment before the draft

Keep the narrative concise, then include the evidence table. The table is mandatory for a normal `处理工单 <id>` investigation and must cover every statement in the draft. Any row that is not `confirmed` is a blocker for the send verdict:

```text
环境/版本：
当前问题：
已确认：
首个失败边界：
根因状态：confirmed | evidence-supported | unknown
历史信息使用：
缺失证据/下一步：
```

| 结论 | 因果角色 | 事实依据 | 环境 / 版本 / 时间范围 | 状态 |
| --- | --- | --- | --- | --- |
| <material claim> | symptom / boundary / trigger / root cause / current state / solution | <ticket message, attachment, exact source ref, applicable document, Linear release evidence, or customer runtime evidence> | <exact scope> | confirmed / evidence-supported / ruled out / unknown |

Do not cite private URLs or confidential content in the customer-facing draft. The internal table may name the evidence system and immutable ref needed for review, but must not reproduce credentials or customer secrets.

## Solution validation standard

Before calling a recommendation a solution, establish and record:

```text
Applicable environment and exact version:
Failure boundary or cause addressed:
Supporting evidence:
Change target and scope:
Prerequisites:
Expected result:
Risk and blast radius:
Rollback or restoration method:
Verification and acceptance evidence:
```

For a production-impacting change, every field above is required and must be supported by the customer's version, applicable documentation/source, and current failure evidence. If the cause is not established or any safety-critical field is unsupported, label the action as a diagnostic check, hypothesis, mitigation, or workaround—not a solution. Do not imply that executing a command proves recovery; require a matching retest of the affected customer path.

## Engineering escalation

Create an escalation only when requested or when support-side investigation establishes a concrete engineering blocker. Keep it internal and do not copy it into the customer draft.

```text
标题：<环境 + 版本 + 首个失败边界>

Zendesk Ticket：#<id>
客户/组织：<name>
环境：<production/test/UAT/development + concrete identifier>
Enterprise/Chart 版本：<raw + normalized>
部署方式：<Helm/Docker/Terraform/custom>

问题与影响：
<customer-visible symptom and scope>

证据判断：
- 首个失败边界：<claim + status>
- 直接触发：<claim + status>
- 根因：<claim + status, or unknown>
- 当前状态：<claim + status>

精确证据：
<timestamps, component, error, relevant config/source refs>

已排除：
<alternatives and evidence>

需要协助：
<specific owner action and acceptance evidence>
```

Do not call an issue fixed because a PR merged. Acceptance should include the target release/ref and a matching environment/user-path verification.
