# xtop-cli developer context

Consolidated developer context for the `xtop-cli` organization. This
repository is derived documentation: it summarizes the eight upstream
repositories, records how they fit together, and packages the material an
AI coding agent or a new contributor needs before touching code.

Upstream repositories remain the source of truth. When this context and an
upstream repository disagree, the upstream repository wins and this
document must be corrected.

## Scope

The context covers exactly these repositories:

| Repository | Role | Context folder |
|---|---|---|
| `xtop` | Kernel: the TUI application and CLI | [`xtop/`](xtop/README.md) |
| `api` | Shared contract crates | [`api/`](api/README.md) |
| `widgets` | Widget packs (renderers) | [`widgets/`](widgets/README.md) |
| `layouts` | Data-driven layouts and layout modes | [`layouts/`](layouts/README.md) |
| `plugins` | Plugin implementations and runtime widget hosts | [`plugins/`](plugins/README.md) |
| `extensions` | Server-style extensions (MCP) | [`extensions/`](extensions/README.md) |
| `effects` | Frame effects | [`effects/`](effects/README.md) |
| `web` | Landing page and documentation site | [`web/`](web/README.md) |

The `web` repository is the only member that is not a Rust crate workspace.
It mirrors the documentation of the other seven repositories and publishes
the public site.

## Checkout model

All documentation assumes a flat multi-repo checkout:

```
~/repos/xtop-cli/
  api/
  effects/
  extensions/
  layouts/
  plugins/
  web/
  widgets/
  xtop/
  context/        <- this repository
```

The context folder is a sibling of the repositories, never nested inside
one of them. Commands in this documentation are written relative to
`~/repos/xtop-cli/<repo>`.

## Reading order

Start with these documents:

1. [`ecosystem.md`](ecosystem.md) - how the repositories depend on each
   other and why the split exists.
2. [`repositories.md`](repositories.md) - inventory, workspace members and
   current state of every repository.
3. [`conventions.md`](conventions.md) - cross-repo engineering rules that
   every change must respect.
4. [`workflows.md`](workflows.md) - development, verification and release
   workflows that span repositories.
5. [`current-state.md`](current-state.md) - dated snapshot of what is
   implemented, what is deferred and which gaps are known.

Agents and skills for automated work live under [`resources/`](resources/README.md).
A compact index for language models is in [`llms.txt`](llms.txt).

## Documentation map

Root level:

- `README.md` - this file.
- `AGENTS.md` - operating rules for AI coding agents.
- `conventions.md` - versions, dependencies, naming, contracts, CI and git
  rules.
- `ecosystem.md` - architecture, dependency graph, integration modes.
- `repositories.md` - repository inventory.
- `workflows.md` - cross-repo workflows.
- `current-state.md` - dated status snapshot.
- `llms.txt` - compact machine-readable index.

Per repository:

- `api/` - contract crates: data model, plugin, widget, extension and
  effect contracts.
- `xtop/` - kernel: architecture, CLI, configuration, theming, providers,
  plugin and extension hosting, operations.
- `widgets/` - widget crate model, widget reference, authoring.
- `layouts/` - JSONC layout schema, authoring guide, shipped defaults.
- `plugins/` - samurai plugin, heuristic rules, WASM and external runtime
  widget hosts.
- `extensions/` - MCP extension and its wire protocol.
- `effects/` - fade effect and effect authoring.
- `web/` - Next.js site, documentation mirroring, search, deployment.

Resources:

- `resources/agents/` - role definitions for coding agents.
- `resources/skills/` - repeatable procedures for common tasks.
- `resources/repos.md` - quick repository-to-path map.

## Maintenance contract

This context is expected to evolve with the ecosystem. Update it when:

- a repository gains or loses a crate, module or documented command;
- a contract in `api` changes in a breaking way;
- a default layout, theme, widget name or plugin action changes;
- a workflow or distribution channel changes;
- the deferred-work list changes.

The maintenance procedure is in
[`resources/skills/context-maintenance.md`](resources/skills/context-maintenance.md).
Do not copy secrets, tokens, private keys, personal contact data or local
machine paths into this repository.

## License

MIT. See [`LICENSE`](LICENSE).
