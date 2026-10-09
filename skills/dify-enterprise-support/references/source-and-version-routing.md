<!-- Public export: deployment-specific identifiers or personal examples were replaced with placeholders on 2026-10-09. -->

# Source and version routing

Use this reference whenever Helm behavior, component compatibility, source implementation, a fixed version, or an upgrade path matters.

## Required source order

1. Read the exact version and affected environment from the current Zendesk ticket or customer evidence.
2. Use `dify-helm-watchdog` to resolve the Enterprise release, app version, component images, release notes, image validation, and upgrade path.
3. Read `dify-helm` at tag `dify-<enterprise-version>` for Chart.yaml, values, templates, and version-to-component mapping.
4. Map the implicated image to its repository and exact tag or commit.
5. Use OpenConnector connection `dify-source` to call `source_search`, `source_read`, `source_log`, or `source_diff` with that explicit ref.
6. Use GitHub search, issues, and pull requests only as supplemental discovery or engineering context.

Never call the source connector with its SaaS `runtime` default for a customer private deployment. Enterprise queries must pass an explicit tag or immutable commit. If the release cannot be mapped to the target repository/ref, label source alignment `unknown`.

## Versioned code operations

Use OpenConnector connection `dify-source` as the primary code connector. Follow [OpenConnector routing](openconnector-routing.md) for discovery and calls:

- `repo_status` verifies that the required repository exists and shows fetch state;
- `refresh_repos` fetches configured repositories when a known tag or commit is missing locally;
- `list_repo_tags` discovers candidate release tags;
- `source_search` searches an exact ref;
- `source_read` reads a file at an exact ref;
- `source_log` inspects the history of a repository or file at an explicit ref;
- `source_diff` compares two explicit refs, optionally for one path.

Every material source result must retain repository, requested ref, resolved commit, file path, and the Enterprise/Chart version that mapped to it. Do not present a result from a branch tip as if it came from the customer's immutable release.

If an exact ref returned by Helm Watchdog is not found:

1. call `repo_status` for the repository;
2. because `refresh_repos` is write-classified, call it once only when the user explicitly authorizes that repository refresh, and include OpenConnector `confirmOperation="write"`;
3. when refresh was authorized and completed, retry the exact ref;
4. if refresh was not authorized or the ref remains unresolved, use an approved GitHub connector or a temporary read-only checkout only when that repository and ref are accessible;
5. otherwise label source alignment `unknown`.

An index miss is not evidence that a feature, configuration, or fix is absent.

## Find when behavior appeared

To identify the first Enterprise version containing a configuration, endpoint, migration, or fix:

1. Search the customer's exact mapped release.
2. Search the candidate newer release identified by documentation, Linear, release notes, or source history.
3. Resolve both releases through Helm Watchdog and search their exact component commits.
4. Compare the adjacent relevant refs with `source_diff`.
5. If claiming "first introduced", verify that the immediately preceding applicable Enterprise release does not contain the behavior.

Report at minimum:

```text
Feature or fix:
First verified Enterprise/Chart version:
Previous version checked:
Repository and exact commit:
File or route:
Evidence status:
```

Absence from one old version plus presence on `main` establishes only a version gap, not the first released version.

## Helm Watchdog

Current MCP endpoint: `https://helm-watchdog.example.com/api/v1/mcp`.

Relevant current tools include:

- `list_versions`
- `get_latest_version`
- `get_version_details`
- `list_images`
- `get_version_release_notes`
- `validate_images`
- `compute_upgrade_path`

Inspect the live tool list because names and schemas may evolve. Prefer Watchdog over scraping the Helm release SPA. Do not recommend "upgrade to latest" before checking supported/unskippable versions and the customer's current state.

## Source repositories

OpenConnector connection `dify-source` exposes these Enterprise product and delivery repositories:

| Repository | Route questions here |
| --- | --- |
| `dify` | API, Web, Worker, Workflow, RAG, model invocation, shared core behavior |
| `dify-enterprise` | Enterprise backend, authentication, members, workspace, audit, gateway, cluster ID, Enterprise plugin management; newer Enterprise frontend under `web/` |
| `dify-enterprise-frontend` | Historical versions with a separate Enterprise administration frontend |
| `dify-sandbox` | code execution, seccomp, dependencies, limits, networking |
| `dify-helm` | release composition, templates, values, image tags, migrations, dependencies |
| `dify-ee-terraform` | cloud infrastructure templates, network, Kubernetes, managed DB/Redis/storage |
| `dify-ee-helm-chart-values-generator` | generated production values and configuration relationships |

Configured plugin repositories outside this seven-repository set:

- `dify-plugin-daemon`: plugin execution, protocol, runtime, installation and invocation boundary;
- `dify-official-plugins`: provider-specific official plugin implementation;
- `dify-plugin-sdks`: Python and Go plugin SDK implementations, protocol contracts, and SDK compatibility.

Plugin routing:

- management, resource, connector, CRD or Enterprise plugin control -> `dify-enterprise`;
- execution, daemon protocol or runtime -> `dify-plugin-daemon`;
- provider behavior -> the exact plugin package/repository;
- SDK contract -> the matching language and SDK version within `dify-plugin-sdks`.

Repository layout is version-dependent. Do not hardcode that Enterprise frontend always lives in a separate repository or always under `dify-enterprise/web`.

## GitHub connector boundary

GitHub code search is useful for default-branch discovery and private issues/PRs, but it is not the primary exact historical source unless the tool supports the required ref. A PR being merged proves repository state only; it does not prove inclusion in a customer's installed release.

## Repositories without release tags

If Terraform, the Values Generator, or `dify-official-plugins` lacks repository release tags, use an immutable commit only when the release, plugin package artifact, manifest/history, or customer evidence maps to it. Otherwise describe findings as current-source guidance and do not claim exact compatibility with the customer's release or installed plugin version.
