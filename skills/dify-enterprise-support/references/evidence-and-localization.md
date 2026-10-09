# Evidence and failure localization

Use this reference for diagnostic, incident, regression, or root-cause claims.

## Choose investigation depth

- `authoritative-answer`: stable product, configuration, release, or compatibility question answerable from an exact-version authoritative source.
- `standard`: one environment/object has a deterministic failure with direct component evidence.
- `deep`: intermittent failures, 5xx/timeout, broad outage, queue pressure, performance degradation, upgrade regression, or unclear multi-component failure.
- `upgrade/migration`: data migration, rollback, supported path, breaking configuration, or unskippable releases.
- `security-sensitive`: authentication, authorization, SSO, CVE, secrets, data exposure, or license material.

Use the lightest route that supports the needed claim. Escalate when the current route cannot establish the first failing boundary.

## Four anchors

A customer-specific causal conclusion needs:

1. exact environment and version;
2. exact object, request path, component, or operation;
3. exact time window and timezone;
4. raw evidence from that path.

Missing anchors must be marked `unknown`, not filled from history or experience.

## Build a sufficient diagnostic evidence set

When the current evidence does not locate the cause, derive the customer request from the remaining execution chain and hypotheses. Ask for every category needed to make the next localization step conclusive; do not use a fixed item count or a generic checklist.

Depending on the unresolved boundary, the evidence set may include:

- affected environment, exact Enterprise/Chart version, deployment method, and relevant component image/tag;
- exact reproduction steps, expected and actual behavior, affected scope, object/request identifiers, and whether the issue is deterministic;
- occurrence time with timezone, first occurrence, last known success, and any change between them;
- raw error and correlated logs from the suspected boundary, scoped to the same identifier and time window;
- effective configuration for the relevant component or feature, including overrides, rather than an unrelated full configuration dump;
- component/Pod/dependency state, events, latency, or resource evidence when the active hypothesis specifically depends on availability, scheduling, timeout, or capacity;
- comparison evidence from another environment or version when diagnosing a regression or environment-specific difference;
- a matching retest result when verifying recovery or a proposed workaround.

For each requested item, record internally:

```text
Requested evidence:
Environment/version/path scope:
Hypothesis or boundary tested:
Expected discriminating result:
Why current evidence is insufficient:
```

Do not ask for an item merely because it is customary. Do not claim a cause until the returned evidence closes the relevant gap.

## Execution-chain method

Reconstruct only the relevant portion of the path, for example:

```text
browser/client
-> DNS / load balancer
-> Ingress / Nginx
-> Web / API / Enterprise service
-> Redis / queue
-> Worker / Plugin Daemon / Sandbox
-> PostgreSQL / vector DB / object storage
-> model provider / IdP / external service
```

Identify the last confirmed success and the first confirmed failure. HTTP 502/504, 401/403, CrashLoopBackOff, timeout, or a browser console error is an initial boundary signal, not automatically the root cause.

Keep these roles separate:

- symptom;
- first failing boundary;
- immediate trigger;
- root cause;
- contributing factor;
- recovery event;
- current state.

## Evidence ledger

For every material live-state or causal claim, record internally:

```text
Claim:
Causal role:
Status: confirmed | evidence-supported | ruled out | unknown
Environment and version:
Object/path:
Time window and timezone:
Evidence source:
Exact observed result:
Alternatives checked:
Remaining gap:
```

Examples of insufficient proof:

- source code without proof that the customer's version contains it;
- a release note without proof the customer completed the upgrade;
- a Pod becoming Ready without a matching user-path retest;
- absence of logs before verifying component, time conversion, retention, and log collection;
- recurrence of a prior error in a different environment;
- a merged PR or assigned issue without released-version evidence.

## Regression claims

Do not call an issue a version regression solely because it appeared after an upgrade. Require a relevant old/new behavior comparison and a configuration, image, schema, or source difference tied to the failure. Otherwise say "首次在升级后观察到" and keep causation `unknown`.
