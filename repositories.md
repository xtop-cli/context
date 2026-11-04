# Repository inventory

Current inventory of the `xtop-cli` organization. All repositories use the
`main` branch. Every Rust repository is MIT licensed; the `web` repository
does not ship a LICENSE file at this snapshot, although the site footer
states MIT. Snapshot date: 2026-11-04.

## Summary

| Repository | Kind | Role | Workspace members | Docs entry |
|---|---|---|---|---|
| `api` | Rust workspace | contracts | 4 crates | `api/README.md` |
| `xtop` | Rust single crate | kernel application and CLI | 1 binary | `xtop/README.md` |
| `widgets` | Rust workspace | widget renderer packs | aggregator + 11 widget crates + core + blocks | `widgets/README.md` |
| `layouts` | Rust single crate | layout model and shipped layouts | 1 crate | `layouts/README.md` |
| `plugins` | Rust workspace | plugins and runtime widget hosts | 6 crates | `plugins/README.md` |
| `extensions` | Rust workspace | server-style extensions | 1 crate | `extensions/README.md` |
| `effects` | Rust workspace | frame effects | 1 crate | `effects/README.md` |
| `web` | Next.js app | site and documentation mirror | not applicable | `web/README.md` |

## api

- URL: `https://github.com/xtop-cli/api`
- Role: shared contracts.
- Members:
  - `crates/plugin-api` -> `xtop-plugin-api`
  - `crates/widget-api` -> `xtop-widget-api`
  - `crates/effect-api` -> `xtop-effect-api`
  - `crates/extension-api` -> `xtop-extension-api`
- Dependencies: `ratatui 0.30.2`, `serde 1`, `serde_json 1`.
- Local CI: `./scripts/ci.sh` with stages `fmt`, `clippy`, `check`, `test`.
- Publication: not on crates.io. Consumers use floating git dependencies.
- Key docs: `docs/architecture.md`, `docs/data-model.md`,
  `docs/plugin-contract.md`, `docs/widget-contract.md`,
  `docs/extension-contract.md`, `docs/effect-contract.md`,
  `docs/changes.md`.

## xtop

- URL: `https://github.com/xtop-cli/xtop`
- Role: kernel application, CLI, providers, state, theme, UI.
- Package: `xtop` version `0.1.0`, binary `xtop`.
- Runtime dependencies: `crossterm 0.29`, `ratatui 0.30.2`,
  `sysinfo 0.39`, `anyhow 1`, `serde 1`, `serde_json 1`, `toml 0.8`.
- Ecosystem dependencies: api contracts, widgets, layouts, and optional
  plugins, extensions and effects.
- Default features: `plugin-samurai`, `mcp-extension`.
- Optional features: `widget-blocks`, `effects`, `plugin-wasm`,
  `plugin-external`.
- Local CI: `./scripts/ci.sh` with stages `fmt`, `clippy`, `check`, `test`,
  `no-default`.
- Structural audit: `./scripts/audit.sh`.
- Automation: `.github/workflows/roadmap-sync.yml` syncs `ROADMAP.md` with
  GitHub issues; `.github/workflows/release.yml` builds multi-platform
  binaries and publishes a GitHub Release on `v*` tags.
- Packaging: `install.sh`, `install.ps1`, `uninstall.sh`, `uninstall.ps1`,
  `PKGBUILD` (AUR `xtop-git`).
- Key docs: `docs/features.md`, `docs/usage.md`, `docs/configuration.md`,
  `docs/customization.md`, `docs/design.md`, `docs/plugin.md`,
  `docs/multi-repo.md`, `docs/colors.md`, `docs/installation.md`,
  `ROADMAP.md`, `CHANGELOG.md`.

## widgets

- URL: `https://github.com/xtop-cli/widgets`
- Role: renderer packs against `xtop-widget-api`.
- Members:
  - `xtop-widget-core` (shared engine, chart, options, formatting, painter,
    `testkit` feature)
  - `xtop-widget-header`, `xtop-widget-cpu`, `xtop-widget-memory`,
    `xtop-widget-storage`, `xtop-widget-network`, `xtop-widget-processes`,
    `xtop-widget-disk_io`, `xtop-widget-battery`, `xtop-widget-gpu`,
    `xtop-widget-summary`, `xtop-widget-sensors`
  - `xtop-widgets` (workspace root aggregator pack)
  - `xtop-widget-blocks` (alternative ASCII pack)
- Names registered by the base pack: `header`, `cpu`, `memory`, `storage`,
  `network`, `processes`, `disk_io`, `battery`, `gpu`, `summary`, `sensors`.
- Community packs: `custom/`.
- Local CI: `./scripts/ci.sh`.
- Key docs: `docs/authoring.md`, `docs/widgets.md`.

## layouts

- URL: `https://github.com/xtop-cli/layouts`
- Role: layout model, JSONC loader, layout modes and shipped layouts.
- Package: `xtop-layout` version `0.1.0`.
- Dependencies: `serde`, `serde_json`; no ratatui and no api dependency.
- Shipped layouts:
  - `layouts/default/` with ten embedded layouts: seven mode-bound plus
    three `detail_*` presets.
  - `layouts/custom/` with community layouts.
- Local CI: `./scripts/ci.sh`.
- Key docs: `docs/layout-schema.md`, `docs/authoring.md`,
  `docs/decisions.md`.

## plugins

- URL: `https://github.com/xtop-cli/plugins`
- Role: plugin implementations and runtime widget hosts.
- Members:
  - `xtop-plugin-samurai` (native plugin, 12 actions, 10 heuristic rules)
  - `xtop-plugin-wasm` (in-process WASM host)
  - `xtop-plugin-external` (helper-process host)
  - `xtop-wasm-contract` (manifest, state and draw-list contract)
  - `xtop-widget-replay` (host-side draw-list replay)
  - `xtop-wasm-guest` (Rust guest SDK with `export_widget!`)
- Examples: `examples/wasm/` (Rust and C guests) and `examples/external/`
  (Lua, Python, Node).
- Local CI: `./scripts/ci.sh` with default stages `fmt`, `clippy`, `check`,
  `test`, and optional stages `wasm`, `external`.
- Key docs: `docs/architecture.md`, `docs/rules.md`,
  `docs/decisions.md` (ADR-001), `docs/wasm-widgets.md`,
  `docs/external-widgets.md`, `CHANGELOG.md`.

## extensions

- URL: `https://github.com/xtop-cli/extensions`
- Role: server-style extensions.
- Members: `xtop-extension-mcp` (MCP server over JSON-RPC 2.0 stdio).
- Compile-time dependency on `xtop-plugin-samurai` for the plugin id and
  action constants.
- Local CI: `./scripts/ci.sh`.
- Key docs: `docs/architecture.md`, `docs/mcp-protocol.md`,
  `extensions/xtop-extension-mcp/README.md`.

## effects

- URL: `https://github.com/xtop-cli/effects`
- Role: frame effects over the rendered ratatui buffer.
- Members: `xtop-effect-fade` (manifest id `fade`, 500 ms fade-in).
- Local CI: `./scripts/ci.sh`.
- Key docs: `docs/effects.md`, `docs/write-effect.md`.

## web

- URL: `https://github.com/xtop-cli/web`
- Role: landing page and documentation site.
- Stack: Next.js 15 App Router, React 19, TypeScript 5.8, `marked`.
- Output: static export (`output: "export"`).
- Languages: Spanish at the root and under `/docs/es/`; English under
  `/en/` and `/docs/en/`.
- Docs source: `docs/` is an English mirror synced from the sibling
  repositories; `i18n/es/docs/` is the Spanish translation.
- Automation: `.github/workflows/deploy.yml` builds and deploys to GitHub
  Pages.
- Key docs: `README.md`, `lib/repos.ts`, `lib/docs.ts`, `lib/i18n.ts`,
  `scripts/sync-docs.sh`, `scripts/build-search-index.mjs`.

## Local checkout and synchronization

```
mkdir -p ~/repos/xtop-cli
cd ~/repos/xtop-cli
for repo in api effects extensions layouts plugins web widgets xtop; do
  [ -d "$repo" ] || git clone "https://github.com/xtop-cli/$repo.git" "$repo"
done
```

Update every checkout:

```
for repo in api effects extensions layouts plugins web widgets xtop; do
  git -C ~/repos/xtop-cli/$repo fetch --all --prune
  git -C ~/repos/xtop-cli/$repo pull --ff-only
done
```

This context repository is a separate repository and is not part of the
loop above.
