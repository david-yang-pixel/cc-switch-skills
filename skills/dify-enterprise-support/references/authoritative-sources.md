<!-- Public export: deployment-specific identifiers or personal examples were replaced with placeholders on 2026-10-09. -->

# Authoritative sources

Use this reference for documentation, release, upgrade, security, compatibility, and customer-link decisions.

## Source priority

Choose the source that can prove the specific claim:

1. current customer evidence for customer-specific state;
2. Helm Watchdog for release/image/upgrade metadata;
3. exact tagged Helm and component source for version behavior;
4. the matching version of official Dify Enterprise Docs for deployment and operation guidance;
5. official Enterprise release information and official Helm documentation;
6. current public Dify documentation and `dify.ai` content read through OpenConnector `dify-docs`;
7. GitHub issues and PRs read through OpenConnector `github-mcp` as supplemental engineering context;
8. historical tickets as contextual leads.

More authoritative documentation does not replace customer-specific runtime evidence.

## Dify Enterprise Docs

`https://enterprise-docs.dify.ai/` is the official Enterprise deployment and operations documentation. It covers deployment preparation, testing and production deployment, Docker Compose, license activation, Helm, high availability, authentication and permissions, databases, Redis, storage, networking, offline deployment, plugins, certificates, performance, and other Enterprise operations.

The root URL normally redirects to the current documentation branch. Paths include an explicit documentation family such as `/en/3.12.x/` or `/en/3.5.x/`. Do not use the root redirect as evidence for an older customer release. Select the closest applicable documented family, record that relationship, and verify material configuration claims against the customer's exact `dify-helm` tag and component refs.

An Enterprise Docs page is normally suitable for customers when it applies to their version and access. Prefer its customer-facing explanation over private source paths or internal release details.

## Enterprise release portal

`https://enterprise-releases.example.com/` is an internal authoritative source for Enterprise release history, release notes, security advisories, LTS/EOL policy, supported upgrade paths, and unskippable versions.

Its Enterprise release information is confidential. Do not put this URL, copied content, or internal release detail into a customer draft by default. Translate only the customer-safe conclusion needed to answer the case. Use a public customer-facing source when one exists.

## Helm release site

`https://langgenius.github.io/dify-helm/#/` is the public Helm release site. Use it for public release history, older release notes, and customer-shareable Helm resources. Because it is a client-rendered application, prefer Helm Watchdog for structured automation and use the site as a human-facing link or cross-check.

## Public Dify documentation and website

Use OpenConnector `custom_mcp` connection `dify-docs` for current public material:

1. search documentation with `official_docs_search` and read exact pages with `official_docs_read`;
2. list, search, and read `dify.ai` website content with `official_site_list`, `official_site_search`, and `official_site_read`;
3. follow `next_offset` until null when later content can change the conclusion;
4. record the source URL, language, refresh/update information, edition, and explicit version scope;
5. verify commands, configuration values, migrations, and compatibility claims against the customer's exact Enterprise version, Helm tag, and component refs.

Current public documentation primarily describes current product behavior. It is discovery evidence, not exact-version proof, when the customer runs an older Enterprise release. Zendesk Help Center is not an official documentation or information source for this skill; do not call `search_help_center` or `get_article` even if those actions are available through the Zendesk connection.

## Other official sources

- Public Dify documentation and website links may be shared when the page directly applies to the customer's edition and version.
- Source code is authoritative for what a specific ref implements, not for what the customer's cluster executed.
- A current document can describe the latest release while the customer runs an older release. Keep answers version-scoped.

When sources conflict, preserve both observations, check their dates and version scope, and mark the unresolved point `unknown` rather than choosing the more convenient answer.

## Versioned documentation method

For every documentation-based claim, record:

```text
Document URL or source:
Document type:
Published or updated date:
Explicit version scope:
Customer Enterprise/Chart version:
Version relationship: exact | older | newer | unversioned
Customer-safe to share: yes | no
```

Classify sources as follows:

- an Enterprise Docs page with a version family in its path is scoped to that documented family, not automatically to every patch or older release;
- an Enterprise release page is scoped to its named Enterprise version;
- a `dify-helm` file is scoped to the exact Chart tag or commit queried;
- generated Helm values and templates at that tag are stronger configuration evidence than a current generic guide;
- an unversioned Dify documentation page describes current behavior unless it explicitly states an older supported range;
- a current page that is newer than the customer's release is discovery evidence only until exact-version source confirms compatibility.

When answering whether a setting or feature applies to a customer version:

1. look for an applicable version family in Dify Enterprise Docs and public Dify documentation explicitly naming the customer's release;
2. inspect the matching `dify-helm` tag and exact component refs;
3. locate the first later release that documents or contains the behavior when absent;
4. compare the relevant adjacent releases;
5. check Helm Watchdog and Enterprise release information for unskippable versions and migrations before giving upgrade guidance.

Never say that documentation is wrong merely because it is newer than the customer's version. State that it documents a different version boundary.
