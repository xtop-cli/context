# Repository operations

Administrative rules for the `xtop-cli` organization. They describe how the
repositories are meant to be operated. They do not grant credentials and do
not contain any.

## Branches and merges

- `main` is the integration branch in every repository.
- Short-lived feature branches are acceptable. The upstream history shows
  merge commits from release and update branches, so a PR-based flow is the
  norm.
- Force pushing `main` is forbidden.
- Do not rewrite published history.

## Pull request expectations

A pull request should state:

- which repository and area it touches;
- whether it changes a contract, a default layout, a theme, a widget name or
  a plugin action;
- which commands were run to verify the change;
- any cross-repository coordination required.

For contract changes, link the consumer updates. A contract change that
lands without its consumers adapted leaves the ecosystem in a state that
does not compile against `main`.

## Issues and roadmap

The kernel repository keeps a `ROADMAP.md` file that is synchronized with
GitHub issues by
`.github/workflows/roadmap-sync.yml` and
`.github/scripts/sync_roadmap.py`.

- Roadmap checkboxes use `#<issue>` references.
- The workflow runs on pushes that touch `ROADMAP.md` and on manual
  dispatch.
- It skips commits whose message contains `[roadmap-sync]` to avoid loops.
- It requires `issues: write` and `contents: write` permissions, provided
  by the workflow token.

Issue triage should distinguish:

- contract changes;
- kernel behavior;
- renderer and layout work;
- plugin and extension work;
- distribution and packaging;
- documentation and website.

## Continuous integration

- Rust repositories: local only. `scripts/ci.sh` is the gate. Do not add
  GitHub Actions to them.
- `web`: GitHub Actions builds the static export and deploys to GitHub
  Pages from `main`.
- `xtop`: the roadmap sync workflow is the only automation.

A merge is acceptable only when the relevant local CI has been run by the
author. Reviewers should not assume a workflow ran.

## Dependencies

- Floating git dependencies are the current intended state. Do not pin a
  revision silently in one consumer while the others float; coordinate the
  change across the ecosystem and document it.
- When the publish phase lands, every consumer must move together.
- Do not add a dependency that contradicts `conventions.md`.

## Releases and packaging

At the current snapshot there are no upstream release tags. Distribution is
source-based plus the AUR `xtop-git` package.

AUR operating rules:

- The package tracks the repository HEAD, so `pkgver` is generated from the
  commit count and short hash.
- Regenerate `.SRCINFO` with `makepkg --printsrcinfo` after editing
  `PKGBUILD`.
- Build the package before pushing. A successful `makepkg` run is the
  minimum verification.
- Commit messages use `upgpkg: xtop-git <pkgver>-<pkgrel>`.
- The AUR repository is separate from the eight upstream repositories.

## Security

- Never commit tokens, keys, credentials or environment dumps.
- GitHub credentials are held by the operator, not by the repositories.
- The MCP server and the plugin system must keep capability enforcement
  intact. A change that bypasses `PluginContext` capability checks is a
  security regression.
- External runtime widgets execute with user privileges. Do not describe
  them as sandboxed.
- WASM runtime widgets are constrained by wasmi fuel and memory limits and
  by a single allowed host import, but they still share the host process.
  Do not describe them as a hard security boundary.
- Report vulnerabilities through the organization's private channel rather
  than a public issue. The repositories do not currently ship a
  `SECURITY.md`; add one before advertising a disclosure process.

## Documentation mirror

The website mirrors Markdown from the upstream repositories. When upstream
docs change:

1. Run `web/scripts/sync-docs.sh`.
2. Update `web/i18n/es/docs/` for the Spanish mirror.
3. Rebuild the site.
4. Verify links and the search index.

A documentation change is not complete until the mirror is refreshed or the
lag is explicitly recorded.

## Context repository

This context repository must remain free of secrets and personal data. It
should be reviewed like code: changes are factual, link to upstream
sources, and are kept in sync with the repository inventory and status
snapshot.
