# Reply Templates

Use this file when drafting customer-facing support replies and the main `SKILL.md` is not enough.

## Contents

- Core writing rules
- Openers and structure
- Signature format
- Scenario patterns
- Short IM-style replies
- Phrases to avoid

## Core Rules

- 起草前后都执行 [reply and handoff](reply-and-handoff.md) 的 Verification gate。下列模板和示例措辞只是表达方式，不是证据：只有对应陈述已达到 `confirmed` 时才能使用（例如"已确认是产品侧问题""已修复""计划在近期版本发布"）；不得用 "looks like"、"可能是" 等推测句代替核实。
- 默认交付两份完整的未发送客户拟回复：英文在前、中文在后；用户明确要求单语或其他格式时从其要求。客户实际发送使用适合客户语言和渠道的版本，不把双语稿合并进一条客户消息。
- 每份拟回复各放在独立的 fenced `text` 代码块中；标题、证据说明和操作提醒放在代码块外，使块内只有可直接复制的正文。
- 如果客户渠道的单条消息长度不足，按语义拆成多条可依次发送的消息；英文各段和对应中文各段分别放在独立 `text` 块中，按目标编辑器显示的长度或字节限制核对。Zendesk 富文本编辑器可能将分隔的 `1.` 段落重排为多个从 1 开始的列表；需要稳定编号时使用 `Q01:` 等普通文本标签。
- Do not mirror the two languages word for word; make each one sound natural.
- Keep English within about 150 words and Chinese within about 200 Chinese characters unless detail is necessary.
- Include exact paths, field names, node names, or endpoints where relevant.
- Customer-facing replies use email/IM tone; internal notes should follow `SKILL.md` mode rules instead.

## Openers

Pick a natural opener based on context:

- Normal: `Hi,`, `Hey,`, `Hi there,`
- Delayed reply: `Hi, sorry for the wait.`
- Follow-up: `Thanks for the details.` or `Got it, let me take a look.`
- Frustrated customer: `Hi, I understand this is frustrating — let me help.`

Chinese equivalents should stay natural and direct, for example:

- `你好，`
- `抱歉让你久等了，我看了一下这个问题。`
- `收到，我先帮你看下。`
- `我理解这会直接卡住你的使用，我来帮你确认一下。`

## Structural Pattern

Default structure:

1. Natural opener (中文必须以"你好，"或等价招呼开头).
2. One or two sentences on what was checked or identified.
3. Concrete next steps or current status.
4. Clear closing instruction.
5. Signature.

## Signature Format (强制)

签名部分 `Best regards,` / `此致` 与 `Dify Support` **必须分两行**，中间用空行隔开：

```
Best regards,

Dify Support
```

```
此致

Dify Support
```

**禁止**写在同一行，如 `Best regards, Dify Support` 或 `此致 Dify Support`。

## Scenario Patterns

### Missing Critical Info

Use when the ticket cannot be diagnosed yet:

- Ask for only the minimum missing data.
- Do not send long questionnaires.

Useful wording:

- `I can help with this, but I need three details first: Workspace ID, the exact error text, and the node/path where it fails.`
- `收到，我可以继续排查；先补充 3 个关键信息：Workspace ID、完整报错、触发路径（例如 Workflow -> Knowledge Retrieval）。`

### Not Resolved Yet

Use when the issue is still under investigation:

- State what is already known.
- Mention the affected component, field, path, or behavior if known.
- Give a realistic update window only if one exists.
- Avoid empty status lines like `We are investigating this.`

Useful wording:

- `I checked this on our side and confirmed the failure is happening in the Knowledge Retrieval node when metadata filtering is enabled.`
- `I reproduced the same error with a minimal workflow on our side, so it is not caused by your workflow configuration.` (only after the reproduction was actually performed)
- `I'll update you as soon as we have confirmation from the engineering team.`

### Known Issue, Fix in Progress

Use when engineering has confirmed the issue and there is a pending fix:

- State that the issue is confirmed.
- State that a fix is prepared or under rollout.
- Avoid exact dates unless release timing is explicitly confirmed internally.

Useful wording:

- `We've confirmed this is a product-side issue and a fix is already in progress. It is planned for an upcoming release.`
- `这个问题我们已经确认是产品侧缺陷，修复已在处理中，预计会在近期版本发布。`

### Fixed

Use when the issue has been resolved:

- Say it has been fixed.
- State whether the customer needs to retry, refresh, republish, or take no action.

Useful wording:

- `This has been fixed on our side. Please rerun the workflow once and it should work normally now.`
- `No action is needed on your end.`

### Calm an Upset Customer

Use when the user is blocked or frustrated:

- Acknowledge the impact in one sentence.
- Move quickly to concrete status or next steps.
- Do not over-apologize or sound scripted.

Useful wording:

- `I understand this is blocking your work — here's what I found.`
- `我理解这会影响你当前的实现，我这边先把已经确认的信息给你。`

## Short IM-Style Reply

Use this when replying in an in-app thread where the customer does not need a full email-like response.

Pattern:

1. One-line acknowledgment.
2. One or two lines on what was found.
3. One next step or what update will follow.

Example:

```text
Hi, I checked this on our side. The error is coming from the backend requiring `metadata_model_config` while the current Cloud UI does not expose that field in this flow.

I've passed these confirmed details to the engineering team and will update you once a workaround or fix is verified.

Best regards,
Dify Support
```

## Avoid

- `We sincerely apologize for any inconvenience this may have caused.`
- `I hope this email finds you well.`
- `尊敬的用户`
- Repeating the same opener every time.
- Vague instructions like `please check your settings`.
