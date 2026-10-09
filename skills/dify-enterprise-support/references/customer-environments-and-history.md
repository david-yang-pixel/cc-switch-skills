# Customer environments and history

Use this reference when prior tickets may clarify a customer's topology, versions, upgrades, customizations, or repeated symptoms.

## One ticket, one current problem

Do not merge multiple historical tickets into the current case. History provides context and hypotheses only. Current-ticket evidence wins when history conflicts.

Search in two scopes when useful:

- requester history, such as `type:ticket requester:<email>`;
- organization history, such as `type:ticket organization_id:<organization_id>`.

Retrieve an index first. Do not load every full ticket for a large organization. Rank candidates, then read roughly 3–5 of the most relevant cases unless more are necessary.

Rank by:

1. same concrete environment, cluster, namespace, domain, or release;
2. same or adjacent Enterprise version;
3. same deployment method and infrastructure;
4. same module, error signature, operation, or object type;
5. proximity to the reported change or incident time.

## Environment timeline

Maintain environment-specific observations rather than a company-global profile:

```text
Organization:
Environment key:
Environment type: production | test | UAT | development | unknown
Cluster/domain/namespace/release:
Deployment method: Helm | Docker Compose | Terraform | custom | unknown
Enterprise version raw:
Enterprise version normalized:
Observed at:
Source ticket:
Confidence: confirmed | evidence-supported | unknown
Customizations/integrations:
Upgrade history:
Prior confirmed incidents:
Last verified state:
```

Build `environment key` from the organization plus a concrete deployment identifier plus environment type. If the concrete identifier is unavailable, keep environments separate by ticket and mark possible matches as hypotheses.

## Reuse rules

Safe to reuse when still applicable:

- known cluster/domain/namespace names;
- deployment method and cloud/provider topology;
- documented custom Ingress, external DB/Redis/object storage, SSO, registry, or air-gapped constraints;
- prior upgrade sequence and previously confirmed incident mechanisms.

Never reuse without current confirmation:

- the affected environment;
- current installed version or image tags;
- whether a previous workaround remains applied;
- current health, root cause, impact, or recovery state;
- credentials, secrets, private URLs, or sensitive configuration.

Historical agent replies and customer beliefs are leads, not proof. Record the ticket and date for every reused fact.
