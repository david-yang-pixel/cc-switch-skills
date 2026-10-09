---
name: dify-enterprise-support
description: 调查 Dify Enterprise/私有部署工单，按客户实际版本核验证据并生成未发送回复。
---

# Dify Enterprise Support

Handle one Enterprise ticket as one current problem. Use customer history to reconstruct environment context and generate hypotheses, never to replace evidence from the current ticket.

## Non-negotiable delivery boundary

- Produce one **customer reply draft**, in Chinese by default unless the user specifies another language or requests an internal-only analysis. Never post, send, or submit a public Zendesk reply or comment, even if a later prompt says to reply directly.
- Never claim that the customer was contacted. State that the draft is unsent when the distinction matters.
- Keep Zendesk and customer systems read-only by default. An internal note, status change, assignment, tag, or other mutation requires a separate explicit request and the configured tool's confirmation gates.
- Do not expose private source repositories, internal URLs, internal discussions, raw credentials, license material, customer secrets, or internal-only release content in the draft.

## Core invariants

- Treat the current ticket's environment and version as authoritative for the current case. A company's production, test, UAT, and development environments may run different versions and topologies.
- Preserve the customer's raw version string, and separately normalize it for comparison. Never turn a company into a single global version record.
- Separate facts, supported hypotheses, and unknowns. Historical tickets, common symptoms, source code, and documentation do not prove what happened in a customer's cluster. Every statement in the customer draft must be `confirmed` under the customer reply verification gate below; otherwise verify it or remove it.
- Do not use Dify SaaS production DB, SigNoz, or SaaS runtime mappings to claim facts about a customer's private deployment.
- Use the exact installed version and component ref whenever a conclusion depends on code or Helm behavior. Do not substitute `main`, a local checkout, or a nearby version.
- When existing evidence cannot locate the cause or first failing boundary, ask for the complete evidence set needed to distinguish the remaining hypotheses and continue diagnosis. Derive each request from a concrete evidence gap; do not impose an arbitrary item limit, and do not append unrelated logs, metrics, configuration, optimization, or capacity-planning requests to an otherwise answerable ticket.

## Workflow

1. Inspect the live tool surface and verify the connectors required for this ticket. Zendesk, Source, GitHub, Linear, and public Dify documentation use the named OpenConnector routes in [OpenConnector routing](references/openconnector-routing.md). Follow [capability preflight](references/capability-preflight.md); a declared dependency is not proof that its tools are callable.
2. Read the current ticket, customer-visible thread, requester and organization, attachments, custom fields, timestamps, and channel through OpenConnector connection `zendesk`. Follow [Zendesk intake](references/zendesk-intake.md).
3. Build a current-case record: affected environment, exact version, deployment method, object/path, symptom, expected behavior, error, first occurrence, timezone, scope, and recent change. Leave missing fields `unknown`.
4. Search requester and organization history only when it can clarify topology, prior upgrades, customizations, or repeated symptoms. Follow [customer environments and history](references/customer-environments-and-history.md).
5. Choose the lightest sufficient route: authoritative-answer, standard diagnosis, deep incident, upgrade/migration, or security-sensitive. Establish the evidence ledger and first failing boundary using [evidence and localization](references/evidence-and-localization.md).
6. For version-sensitive claims, start with Helm Watchdog and `dify-helm`, map the release to component refs, then query OpenConnector connection `dify-source` at explicit refs. When the question is when behavior appeared or changed, compare adjacent mapped releases rather than searching only the customer version and `main`. Follow [source and version routing](references/source-and-version-routing.md).
7. Use the matching version of Dify Enterprise Docs, the Enterprise release portal, official Helm material, and OpenConnector `dify-docs` for applicable public Dify documentation and website content according to [authoritative sources](references/authoritative-sources.md). Zendesk Help Center is excluded. A current or unversioned page does not automatically apply to an older Enterprise release. Use [deployment troubleshooting](references/deployment-troubleshooting.md) only when customer-side runtime or infrastructure evidence is needed.
8. When the symptom may be an existing defect, regression, limitation, or released fix, search Linear through OpenConnector provider `linear`, connection `default`, and verify the full issue, comments, relations, and release evidence. Follow [known issues and fixes](references/known-issues-and-fixes.md).
9. If new customer input arrived or the investigation was long enough for the ticket to change materially, refresh the thread and relevant attachments. Reuse a still-current intake otherwise. Produce the requested assessment and unsent draft according to [reply and escalation](references/reply-and-escalation.md).

## Evidence labels

Use these labels internally for every material live-state or causal claim:

- `confirmed`: direct evidence proves the claim for the correct environment, object, version, and time.
- `evidence-supported`: the available chain supports the claim and stronger alternatives were checked.
- `ruled out`: relevant evidence contradicts the hypothesis.
- `unknown`: evidence is missing, stale, contradictory, or from a neighboring environment/path.

Keep symptom, first failing boundary, immediate trigger, root cause, contributing factor, recovery event, and current state separate. Only `confirmed` claims may appear as statements in the customer draft; `evidence-supported` and `unknown` claims must be verified first or removed.

## Customer reply verification gate

Customers receive only statements backed by conclusive evidence or verification. Apply this gate to every customer draft, including short replies, follow-ups, and revisions. Details are in [reply and escalation](references/reply-and-escalation.md) under the evidence-grounded output gate.

- Each statement (fact, cause, current state, configuration or command, version compatibility, third-party behavior, support scope, recommendation, or promise) must be `confirmed` by evidence read in this investigation. Acceptable evidence: the customer's own runtime evidence, exact-version component source or a rendered chart, applicable versioned documentation, the vendor's official documentation, explicit current confirmation from the responsible owner, or a verification actually performed.
- Inferences, historical tickets, AI-generated notes, prior replies, internal comments, and memory are leads. Statements taken from documentation or prior replies must also be checked against mechanism and version scope before reuse.
- If a statement cannot be verified but the customer needs to know the gap, write only a true status and list it as a blocker. Never fill a gap with a plausible answer.
- Configuration that was verified only by source or rendering, not in a running environment, must be presented as such and the customer asked to validate it in a test environment first.
- Return a send verdict with every draft: `可发送` only when every statement is `confirmed` and no blocker remains; otherwise `暂不可发送`, with each unverified statement, the missing evidence, and who or what can confirm it.

## Default result

Return:

1. A concise Chinese internal assessment with environment/version, failure boundary, hypotheses, and missing evidence.
2. A mandatory internal evidence table mapping every statement in the customer draft to its evidence, environment/version/time scope, and status.
3. One customer-facing draft in a fenced `text` block, in Chinese unless the user specifies otherwise. Omit the draft for an explicitly internal-only task.
4. A send verdict (`可发送` or `暂不可发送`) with any blockers.

If the user explicitly asks only for reply wording, omit the internal assessment and evidence table, but still enforce the verification gate before drafting and still return the send verdict and blockers.

Do not create a local ticket draft file. Do not add English unless the user explicitly requests it.
