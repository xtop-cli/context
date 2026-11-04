# AGENTS.md

Operating instructions for AI coding agents working with the `xtop-cli`
ecosystem and this context repository.

## Precedence

1. The upstream repository being changed is authoritative.
2. The contracts in `~/repos/xtop-cli/api` are authoritative for every
   type that crosses a repository boundary.
3. This context repository is derived documentation. It explains and
   indexes; it does not override source code.
4. If code and documentation disagree, stop, verify against the code, and
   correct the documentation rather than silently following the stale text.

## Ground rules

- Work in English. The only non-English content in the ecosystem is the
  Spanish documentation mirror under `web/i18n/es/docs/` and some historical
  changelog entries.
- Do not add emojis to code, documentation, commit messages or issue text
  unless a human explicitly requests them.
- Never write secrets, tokens, SSH keys, credentials, personal email
  addresses or absolute local paths into this context repository.
- Do not invent facts. Every architectural claim must be traceable to a
  file, doc or command in an upstream repository.
- Do not add speculative API surface. The ecosystem deliberately grows
  contracts only when a real consumer exists.
- Respect the version policy: every crate stays at `0.1.0` for now. Do not
  bump versions unless the human explicitly requests a release.
- CI is being introduced gradually. The Rust repositories keep local CI as
  their gate (`scripts/ci.sh`); do not add a new workflow without an
  explicit decision. Current intentional workflows: `xtop` has the
  roadmap-to-issues sync and a tag-triggered `release.yml`; `web` has the
  Pages deployment workflow.
- Do not commit generated build output, `Cargo.lock` files in library
  workspaces, `node_modules`, `target/`, `pkg/` or `out/`.

## Checkout assumptions

The repositories are siblings:

```
~/repos/xtop-cli/
  api/  effects/  extensions/  layouts/  plugins/  web/  widgets/  xtop/
  context/
```

All commands must use this layout. If a checkout differs, state the actual
layout explicitly instead of hardcoding it into documentation.

## Repository ownership map

| Area | Repository | Contract dependency |
|---|---|---|
| Data model, plugin protocol, widget render contract, extension protocol, effect protocol | `api` | none |
| Kernel application, CLI, providers, state, theme, UI engine, plugin hosting | `xtop` | `api`, `widgets`, `layouts`, `plugins`, `extensions`, `effects` |
| Widget renderers | `widgets` | `api` |
| Layout model and loader | `layouts` | none |
| Plugin implementations, WASM/external runtime hosts | `plugins` | `api` |
| Server-style extensions | `extensions` | `api`, `plugins` (MCP depends on samurai constants) |
| Frame effects | `effects` | `api` |
| Site, docs mirror, search | `web` | none at compile time |

## Required reading per task

- Kernel work: `xtop/README.md`, `xtop/architecture.md`,
  `xtop/operations.md`, and the relevant module doc.
- Contract change: `api/architecture.md`, the affected contract document,
  `api/changes.md`, and `conventions.md`.
- Widget work: `widgets/architecture.md`, `widgets/widget-reference.md`,
  `widgets/authoring.md`, `api/widget-contract.md`.
- Layout work: `layouts/schema.md`, `layouts/authoring.md`,
  `layouts/defaults.md`.
- Plugin or runtime widget work: `plugins/architecture.md`,
  `plugins/runtime-widgets.md`, `plugins/wasm-widgets.md`,
  `plugins/external-widgets.md`.
- MCP work: `extensions/mcp-protocol.md`, `extensions/architecture.md`.
- Effect work: `effects/architecture.md`, `effects/catalog.md`,
  `effects/authoring.md`.
- Web work: `web/architecture.md`, `web/content.md`, `web/operations.md`.
- Release or packaging work: `xtop/operations.md`, `workflows.md`,
  `resources/skills/release-core.md`.

## Change procedure

1. Confirm the current state of the affected repository:
   `git -C ~/repos/xtop-cli/<repo> status`, `git log -1`.
2. Read the relevant context documents and the upstream source.
3. If the change crosses repositories, start with `api`. Temporary path
   dependencies or `[patch]` sections are allowed during development, but
   every manifest must be restored to its floating git dependency form
   before anything is pushed.
4. Implement the smallest complete change. Match existing style, module
   boundaries and documentation tone.
5. Run the repository's local CI. Never claim a check passed without
   running it.
6. Update this context repository when the change alters documented
   behavior.
7. Report exactly what changed, what was verified and what remains
   unverified.

## Verification commands

```
# API contracts
cd ~/repos/xtop-cli/api && ./scripts/ci.sh

# Kernel
cd ~/repos/xtop-cli/xtop && ./scripts/ci.sh
cd ~/repos/xtop-cli/xtop && ./scripts/audit.sh

# Widgets
cd ~/repos/xtop-cli/widgets && ./scripts/ci.sh

# Layouts
cd ~/repos/xtop-cli/layouts && ./scripts/ci.sh

# Plugins (default stages)
cd ~/repos/xtop-cli/plugins && ./scripts/ci.sh

# Plugins runtime host stages (extra toolchains)
cd ~/repos/xtop-cli/plugins && ./scripts/ci.sh wasm
cd ~/repos/xtop-cli/plugins && ./scripts/ci.sh external

# Extensions
cd ~/repos/xtop-cli/extensions && ./scripts/ci.sh

# Effects
cd ~/repos/xtop-cli/effects && ./scripts/ci.sh

# Web
cd ~/repos/xtop-cli/web && npm ci && npm run build
```

## Forbidden actions

- Editing or deleting upstream history.
- Force pushing any repository.
- Committing generated `target/`, `pkg/`, `out/` or `node_modules/` trees.
- Adding a dependency that violates the ecosystem dependency policy
  (`conventions.md`).
- Copying the implementation of a contract type into a consumer.
- Reporting a warning or a skipped check as a pass.
- Adding personal data or machine-specific paths to documentation.

## Output expectations

When a task completes, report:

- repositories and files touched;
- contract or behavior changes;
- commands executed and their result;
- follow-up work that remains, if any.

Keep the report short and factual. Do not include speculative roadmap items
as if they were implemented.
