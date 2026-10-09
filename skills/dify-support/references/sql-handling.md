# SQL Handling

Use for SaaS production-data verification.

## Route and databases

Execute read-only SQL through OpenConnector `custom_mcp` connection `dify-db`, discovered action `db_gateway_query`.

- `prod`: application, workspace, account, run, dataset, document, and TiDB binding state.
- `billing`: subscription, Stripe mirror, feature/quota, education/promotion state.
- `plugin`: plugin installation/runtime state.

Inspect live `information_schema` before using uncertain tables or columns. Do not rely on local schemas or remembered fields.

Map account email → account → tenant/workspace in `prod`, then use the verified tenant ID in `billing` or `plugin`. Treat a missing Stripe mirror separately from promo/manual entitlement. For vector storage, map only to `prod.tidb_auth_bindings.cluster_name`; TiDB Cloud Web supplies storage facts.

## Working rules

- Replace placeholders with the ticket's verified identifiers.
- Label the database used and explain what each result proves.
- When a result is unexpected, run the smallest next verification query rather than stopping at one snapshot.
- Keep current state separate from reported-time history for billing, quota, reserve, retry, and async cases.
- Never execute INSERT, UPDATE, DELETE, DDL, procedures, locks, or other mutations. Route repair to an authorized Admin/SRE/engineering workflow.

Do not expose raw SQL, private identifiers, or internal table names in customer-facing drafts.
