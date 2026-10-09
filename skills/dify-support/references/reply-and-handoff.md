# Reply And Handoff

## Verification gate

Customers receive only statements backed by conclusive evidence or verification. Run this gate before presenting any customer draft, including short replies, follow-ups, and revisions.

1. Split the draft into atomic statements: facts, causes, current state, configuration or commands, version and compatibility claims, third-party behavior, policy or support scope, recommendations, and promises.
2. Every statement must be `confirmed` by evidence read in this investigation:
   - customer-specific state or cause: the customer's own object evidence (DB, logs, trace, attachment) for the right object, path, version, and time window;
   - product behavior or configuration: source at the aligned runtime ref, rendered or observed configuration, or current authoritative documentation for that environment;
   - third-party behavior (cloud, model provider, plugin vendor, database): that vendor's official documentation or directly observed behavior;
   - policy, plan, refund, support scope, or ETA: a current authoritative policy source or explicit confirmation from the responsible owner in the current period;
   - fix or workaround: the validation level actually reached under [recovery validation](recovery-validation.md); state customer-side validation as still required when the path was not verified.
3. `evidence-supported` claims, inferences, historical tickets, AI-generated notes, prior agent replies, memory, and typical behavior are leads. Verify them or remove them from the draft.
4. Documents, historical replies, and internal comments can be wrong, outdated, or out of scope. Before reusing a statement from them, check its mechanism, version scope, and applicability to this case.
5. If a statement cannot be verified but the customer needs to know the gap, write only a true status (for example, that the point is being confirmed with engineering) and list it as a blocker. Never fill a gap with a plausible answer.
6. Check absolute or general wording (`only`, `all`, `never`, `not supported`, `fixed`, `will`): its evidence must cover the full scope; otherwise narrow the statement.
7. After every edit, re-read the whole draft sentence by sentence; new wording must pass the same gate.

Deliver with every draft:

```text
逐句核实：
| 客户草稿中的陈述 | 证据（来源 / 对象 / 版本 / 时间） | 状态 |
发送结论：可发送 | 暂不可发送
阻塞项：<未核实的陈述 → 缺少的证据 → 确认途径或负责人>
```

Mark `可发送` only when every statement is `confirmed` and no blocker remains. Otherwise do not call the draft final, ready, or sendable.

## Customer draft

- Follow the user's requested language and channel; otherwise match the customer's language. Read [reply templates](reply-templates.md) for opener, signature, length, and formatting.
- Every statement must pass the verification gate above; live-state and causal statements must also map to the evidence ledger.
- State verified facts first. If root cause remains unknown, explain the checked boundary and minimum missing data without implying certainty.
- Give exact customer-safe steps and the validation level of any workaround. Do not promise ETA, refund, approval, deployment, or recovery without evidence and authority.
- Do not expose internal SQL, logs, private links, IDs, infrastructure details, credentials, or source snippets.
- A prepared draft is unsent unless the external system readback proves delivery.

## Agent reply review

Re-read the latest conversation first. Split the reply into claims and mark each `correct`, `incomplete`, `misleading`, `unsupported`, or `customer-unsafe`. Check environment, app mode, node, plugin/provider version, object, time, and execution path. Recommend no follow-up, clarification, correction, or the minimum additional evidence. Any claim that is not `correct` blocks sending until it is corrected and passes the verification gate.

## Engineering/Linear draft

Use concise Chinese unless requested otherwise:

```text
标题：<一句话标题>

用户信息：
Zendesk Ticket: #<id>
Workspace/Tenant ID: <id>
App/Workflow/Dataset/Document/Trace ID: <ids>

问题：
<现象和影响>

因果判断：
影响范围：<object-specific/tenant-specific/multi-tenant-platform/unknown，含样本和窗口>
首个失败边界：<confirmed/evidence-supported/unknown>
直接触发条件：<confirmed/evidence-supported/unknown>
根因：<证据支持的根因；否则写当前证据不足>
当前状态：<已恢复/仍异常/unknown，以及证据>

已确认信息：
<对象、UTC 时间、版本/路径和证据>

恢复方案：
<候选、验证级别、结果、风险和剩余检查>

已有工程记录：
<Linear/讨论/PR/发布/部署的精确证据级别>

需要协助：
<owner、动作和需要回传的证据>
```

If the ticket reveals a concrete product, SRE, monitoring, SOP, or automation improvement, add a short evidence-backed engineering suggestion. Omit speculative improvements.
