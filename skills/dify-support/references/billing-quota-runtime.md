# Billing, Quota, And Runtime State

Use this reference for SaaS tickets involving quota, workflow reserve, rate limit, billing downgrade/upgrade, credits, subscription status, Stripe payment, or "plan looks active but feature is blocked".

## Contents

- Core rule and execution chain
- Minimum evidence and DB/SigNoz checks
- Stripe and billing timeline
- SRE/owner handoff
- Customer-safe wording

## Core Rule

Do not decide from the current DB snapshot alone. These cases need two answers:

1. Why did the customer see the error at the reported time?
2. Is the workspace currently recovered?

`tenant_feature_usages.usage` is not realtime quota truth. It may lag behind runtime quota/cache state.

## Chain To Verify

Typical chain:

```text
Stripe invoice/payment -> billing webhook -> subscription state -> plan/feature quota -> runtime quota/cache -> reserve/check -> product path error -> post-fix success
```

Possible runtime quota/cache keys include quota `q/u/r` style keys. If support tools cannot read them, ask SRE/owner for the exact runtime state instead of replacing it with DB usage.

## Minimum Evidence

| Hop | Evidence |
| --- | --- |
| Customer error | Zendesk timestamp, screenshot/error, workspace ID, app/workflow/run ID |
| Current plan | billing DB `tenant_subscription_settings`, subscription detail/status tables |
| Stripe/payment timeline | invoice, payment attempts, paid/failed timestamps, subscription `past_due`/`active` transitions, webhook event IDs if exposed |
| Plan/quota refresh | billing sync, webhook handling, downgrade/refresh cron logs or records |
| Runtime quota/cache | Redis/runtime quota keys, quota info endpoint, reserve/check logs, rate-limit counters |
| Product rejection | SigNoz logs/traces for reserve failure or feature limit error in the affected service/worker |
| Recovery | successful workflow/API runs after fix, current effective plan/quota, cleared runtime counters when applicable |

## DB Checks

Use `dify_db` MCP:

- `database="billing"` for subscription, Stripe status, feature usage, education/promo state.
- `database="prod"` for workspace, apps, workflow_runs, workflow_node_executions, credit pools, conversations/messages.
- `database="plugin"` only if plugin quota/runtime is involved.

For quota/billing questions, never rely only on:

- `tenant_feature_usages.usage`
- current `tenant_subscription_settings.plan`
- current `tenant_credit_pools`
- workspace UI status

Use them as one hop in the chain, then verify runtime and timeline evidence.

## SigNoz Checks

Search the customer error window first:

- tenant/workspace ID
- app/workflow/run ID
- exact error text
- feature_key, quota, reserve, rate_limit, billing, subscription, past_due, plan
- worker deployment if workflow/dataset path is async

For workflow quota errors, check both:

- the customer error window for rejection logs; and
- later successful workflow_runs to confirm recovery.

## Stripe/Billing Timeline

For payment-related cases, reconstruct absolute UTC timestamps:

1. subscription active period and renewal date;
2. invoice created;
3. automatic payment failure/success;
4. retry attempts;
5. webhook event handling;
6. downgrade or refresh cron;
7. manual payment or recovery;
8. subscription state transition back to active;
9. product runs after recovery.

If Stripe event tools are not available, state the gap and ask SRE/billing owner with invoice/subscription IDs and the time window.

## SRE/Owner Ask Template

```text
Please help confirm quota/runtime state for tenant <tenant_id> during <UTC window>.
Verified:
- billing DB: <plan/status/expiration/current_period facts>
- Stripe/billing timeline: <known invoice/payment/webhook facts>
- product DB: <workflow/API error and post-recovery run facts>
- SigNoz: <reserve/error logs found or missing>
Missing:
- runtime quota/cache key state, especially q/u/r or quota info at that time
- whether downgrade/refresh cron cleared or retained runtime counters
Need:
- effective quota and usage at <error time>
- reserve/check rejection reason
- recovery event that cleared/restored runtime state
```

## Customer-Safe Wording

Do not expose Stripe IDs, Redis keys, internal cron names, DB table names, or SigNoz details to the customer unless the user explicitly asks for an internal report.

Customer reply should say only:

- what status was confirmed;
- whether the issue is fixed now;
- whether they need to retry or take action;
- whether investigation is still pending because an internal runtime state must be verified.
