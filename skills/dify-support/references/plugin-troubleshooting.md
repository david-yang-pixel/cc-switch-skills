# SaaS Plugin Troubleshooting

Use for Dify Cloud plugin installation, invocation, provider, connector, or runtime failures.

## Evidence order

1. Refresh ticket details, exact error, timestamp, workspace/app/run/node, plugin/provider name, and customer action.
2. Through `dify-db` `plugin`, verify `plugin_installations.plugin_unique_identifier` and the installed package/version for the workspace.
3. Check Marketplace current package/version when upgrade behavior matters.
4. Through `dify-signoz`, locate plugin-daemon/connector services, aggregate scope, inspect tenant/plugin logs and matching traces, and identify the first failing boundary.
5. Through `dify-source`, align the deployed plugin daemon/connector runtime and inspect source at the resolved ref. Inspect plugin package source for the installed version when explaining current behavior.
6. Compare another call/object only when needed to distinguish workspace configuration, one plugin, shared runtime, or provider outage.

Do not diagnose from symptom wording alone. A timeout can originate in connector networking, runtime capacity, plugin code, or downstream provider. An installation failure can originate in package/version, compatibility, download/build, or workspace state.

Keep conclusions version-scoped: “the installed version does not include X” is different from “Dify does not support X.” For a third-party plugin defect, direct the customer to the Marketplace author/repository without promising Dify will coordinate; Dify-maintained plugins may receive an engineering handoff.

Enterprise/private-deployment plugin cases belong to `$dify-enterprise-support`.
