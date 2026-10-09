# Deployment troubleshooting

Use this reference when the answer requires customer-side Helm, Kubernetes, Docker, infrastructure, database, storage, network, or runtime evidence.

## Establish scope before commands

Confirm environment, namespace, Helm release, Enterprise version, occurrence time/timezone, affected component, expected behavior, and recent change. Keep production and non-production evidence separate.

Ask for a sufficient and safe evidence set that can distinguish the remaining hypotheses and locate the failing boundary. Prefer commands that read state and limit output by namespace, component, and time.

Common safe examples, adapted to the customer's names:

```text
helm list -n <namespace>
helm get values <release> -n <namespace> -a
kubectl get pods -n <namespace> -o wide
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace> -c <container> --since=<duration>
kubectl get events -n <namespace> --sort-by=.lastTimestamp
```

Request only the relevant values sections and logs. Ask customers to redact secrets, tokens, passwords, internal domains, personal data, license content, and registry credentials.

## Symptom routing

### Installation or upgrade

Check the supported upgrade path, backups, migration Jobs, Helm hooks, image availability, values changes, deprecated keys, and component readiness. A successful Helm command does not prove data migration or the user path succeeded. For rollback advice, distinguish application rollback from database/schema restoration.

### Image pull, offline, or architecture

Use Watchdog image validation, then check the exact repository/tag, CPU architecture, registry mirror, imagePullSecrets, pull error, and node architecture. `ImagePullBackOff` identifies the image boundary; the raw event determines whether the trigger is missing image, authentication, rate limiting, DNS, TLS, or architecture.

### 502, 503, 504, timeout, or connection reset

Trace load balancer -> Ingress -> Service -> endpoint/Pod -> application dependency. Collect the exact host/path/method/time/status and matching Ingress/application logs. Do not assume an Ingress timeout when the upstream application or dependency failed first.

### 401, 403, SSO, or login

Separate browser/session, callback URL, reverse-proxy headers, clock, IdP configuration, Enterprise auth service, account state, workspace permissions, and license state. Never request private keys or complete tokens.

### CrashLoopBackOff or readiness failure

Check the failing container, last termination reason, exit code, events, startup logs, probes, configuration mounts, resource limits, architecture, and dependency availability. A restart count is impact evidence, not a cause.

### Worker, workflow, queue, or plugin failure

Identify the exact workflow/run/node/plugin/version and correlate API, Worker, Redis/queue, Plugin Daemon, Sandbox, provider, and callback boundaries. A UI timeout may coexist with background execution; verify final run state.

### Database, Redis, vector DB, or object storage

Confirm whether the dependency is embedded or external, its endpoint/TLS/auth mode, pool/limit state, migration version, error timestamp, and the first Dify component reporting failure. Avoid destructive SQL, broad dumps, and production writes.

### Performance or intermittent failure

Establish affected population, latency/error baseline, resource saturation, concurrency, queue depth, Pod distribution, and whether test and production share dependencies. A single slow request does not prove cluster-wide capacity exhaustion.

## Stop and escalate

Stop speculative command expansion when:

- the affected environment/version is still unknown;
- requested evidence would expose secrets or excessive customer data;
- a destructive repair, database mutation, or unsupported rollback would be required;
- source-to-release alignment is unresolved;
- the issue requires engineering ownership rather than another customer command.

State the exact unresolved boundary and the evidence set or owner action required to continue localization.
