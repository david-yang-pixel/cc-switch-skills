# Refunds

Use this file for refund tickets. Refund tickets use the Admin operations platform by default. Normal refunds should not default to manual Stripe or finance-team execution.

## Contents

- Execution order and decision rules
- Customer reply requirements
- Full and pro-rated refund guidance
- Rejection and pending-verification guidance
- Admin operation note template

Default outputs are:

1. Customer reply.
2. Admin operation note or exception-escalation note, only after owner confirmation.

## Execution Order

Use this order:

0. Verify the requester is the workspace owner. If not, stop and ask for owner account/email confirmation before any eligibility judgment.
1. Quick retention attempt (when suitable and only after owner verification).
2. Refund eligibility decision.
3. Customer confirmation.
4. Admin refund ticket: refresh facts, choose current or historical invoice, then execute full or partial refund only after explicit confirmation.
5. Check Admin ticket timeline/refund status and reply to the customer.

Retention attempt is suitable when the customer reports product issues shortly after purchase and asks for refund first. Keep it short and practical, then proceed with refund if they still want it.

Useful retention wording:

- `If you’re open to it, I can prioritize a quick fix check on your blocking issue first. If it still doesn’t work for your use case, I’ll proceed with the refund right away.`
- `如果你愿意，我可以先帮你快速处理当前卡点；如果仍不满足你的使用需求，我会马上帮你走退款。`

## Decision Rules

- Full refund: renewal/new purchase is within 14 days and there was no usage after renewal.
- Pro-rated refund: renewal is 14-30 days ago, or there was small post-renewal usage such as apps, member changes, storage, or other billable usage; handle through Admin by default, escalate only for amount, policy, permission, platform failure, or legal/compliance exceptions.
- Reject refund: more than 30 days have passed after renewal and there was substantial usage.
- Pending verification: usage evidence is missing, conflicting, or not queryable yet.

Always verify the requester is the workspace owner before judging eligibility. If the requester is not the owner, stop and ask for owner account/email confirmation.

## Customer Reply Requirements

- State what was checked in the subscription or billing status.
- State whether the case qualifies for full refund, pro-rated refund, or rejection.
- For refundable cases, ask the customer to confirm before the refund is submitted.
- Mention the refund usually arrives within 5 to 10 business days.
- For pro-rated refunds, say the subscription will be canceled immediately after the refund is processed.
- For pending verification, clearly state what data is still needed and when you will update.

## Full Refund Reply Guidance

Include:

- The subscription or renewal that was checked.
- Confirmation that there was no usage after renewal.
- That a full refund can be processed.
- A request for confirmation from the customer.
- The 5 to 10 business day timeline.

## Pro-Rated Refund Reply Guidance

Include:

- The subscription or renewal that was checked.
- That usage was detected after renewal, so full refund is not available.
- Billing period, used days, remaining days, or refund basis if available.
- A request for confirmation from the customer.
- The 5 to 10 business day timeline.
- That the subscription will be canceled immediately after the refund.

## Rejection Guidance

Include:

- What was checked.
- Why the case does not qualify under the refund rule.
- If possible, offer an alternative such as plan downgrade or cancel-at-period-end.

## Pending Verification Guidance

Use when database records are inconsistent or cannot confirm usage yet:

- Tell the customer verification is in progress with specific missing evidence.
- Do not promise a refund type yet.
- Provide a concrete next update window if possible.

Useful wording:

- `I’ve confirmed your subscription is active, but we still need to verify post-renewal usage to determine whether this is a full or pro-rated refund. I’ll update you once the billing check completes.`
- `我已确认你的订阅状态正常，但还需要核对续费后的使用记录，才能判断是全额还是按比例退款。我会在账单核对完成后第一时间更新你。`

## Admin Operation Note Template

Use this template exactly and fill in the values:

```text
**退款处理记录 — [用户姓名]**

- **Workspace ID**: xxx
- **用户邮箱**: xxx
- **计划**: xxx
- **最近扣费时间**: xxx
- **退款类型**: 全额退款 / 按剩余天数折算退款
- **计费周期**: [开始日期] – [结束日期]（共 [总天数] 天）
- **已用天数**: [已用天数] 天
- **原因**: 一句话说明退款原因
- **Admin Ticket**: xxx
- **执行结果**: succeeded / failed / needs approval
```

Rules:

- Omit `计费周期` and `已用天数` for full refunds.
- Do not include Stripe customer ID or Stripe subscription ID.
- Do not manually operate Stripe for normal refunds; use the Admin operations platform.
- Escalate only when amount/policy/permission/platform failure/legal risk requires it.
