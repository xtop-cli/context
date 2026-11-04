# Current state

Dated snapshot: 2026-11-04. This document records what is implemented,
what is deliberately deferred and which gaps are known. Treat it as a
status report, not as a specification. Verify against the repositories
before acting on it.

## Versions

- Every Rust crate reports `0.1.0`.
- Kernel commit at snapshot: `cb17bb7` on `main` (35 commits).
- AUR package `xtop-git` at snapshot: `r35.cb17bb7-1`.
- No upstream repository has a release or tag at the snapshot except the
  local AUR packaging repository, which has its own history.
- Website package `xtop-web` is private version `0.1.0`.

## Implemented

### Contracts (`api`)

- Four crates: plugin, widget, extension and effect APIs.
- Shared data model with 22-field `ProcessInfo`, snapshot lifecycle and the
  `AlertThresholds` serde contract.
- Capability-gated plugin context with `Result` reads.
- Canonical glyph helpers shared by all widget packs.
- Docker model surface removed.

### Kernel (`xtop`)

- Single-crate binary organized into `commands`, `config`, `plugins`,
  `providers`, `state`, `theme` and `ui`.
- CPU, memory, swap, storage, disk I/O, network, process, battery and GPU
  monitoring with platform probes on Linux, macOS and Windows and honest
  empty states elsewhere.
- Twelve themes in format v2 with WCAG contrast normalization.
- Seven layout modes plus three Detail presets, user and community layouts.
- Widget packs with per-widget and per-instance options.
- Command palette, help overlay, process search, sorting with direction,
  process termination and fullscreen mode.
- Plugin system with capability enforcement and error isolation.
- MCP extension exposed as `xtop mcp`.
- Optional effects and runtime widget hosts.
- CLI theme switching (`xtop --ct <theme>`) that running instances follow.
- Local CI and structural audit scripts.
- Roadmap-to-issues synchronization workflow.
- Install and uninstall scripts for Linux and Windows.
- AUR `xtop-git` packaging.

### Ecosystem

- Samurai plugin with 12 JSON actions and 10 heuristic rules.
- MCP server exposing the samurai actions as 12 tools.
- Fade effect, deterministic and stateless.
- WASM runtime widget host with fuel, memory and import limits, hot reload
  and a draw-list contract.
- External helper-process host with line-delimited JSON and per-response
  timeouts.
- Base widget pack with 11 widgets and an alternative blocks pack.
- Ten embedded layouts including dense detail presets.
- Web site with mirrored English docs, Spanish translations, search index
  and GitHub Pages deployment.

## Deferred or not implemented

Contract and distribution:

- The four `api` crates are not published on crates.io.
- Consumer git dependencies are floating, not pinned to tags or revisions.
- Repository tags and GitHub releases do not exist upstream.
- Per-repo CI is local only; release automation is not implemented.

Kernel and ecosystem:

- Native dynamic plugin discovery (ABI or directory-based loading of
  compiled plugins) is deferred.
- Hook-style extensions (pre/post render hooks, config transforms) are not
  designed or implemented. Only server-style extensions exist.
- Docker container metrics were removed from the data model and are not
  planned until a real consumer exists.
- Per-process detail panels, tree process view, follow mode and per-core
  temperature coverage beyond Linux are deferred with documented reasons.
- Transparent background and 16-color terminal downconversion are deferred.
- Mouse input is limited to wheel scrolling of the process list.
- `xtop-git` is the only official package in this organization's
  distribution scope at the snapshot; Homebrew and Winget formulas are not
  present in these eight repositories.

## Known gaps and risks

- Floating git dependencies mean a sibling change can reach consumers on
  the next fetch without a version bump or tag.
- The kernel audit does not implement module dependency-cycle detection or
  unused-public-item detection.
- The `web` docs mirror is a manual sync. A stale mirror is possible until
  `scripts/sync-docs.sh` is run and the site is rebuilt.
- Widget ids in layouts are not validated at parse time; `xtop layout
  check` validates structure only.
- External runtime widgets are not sandboxed beyond the process boundary.
- The Spanish documentation mirror can lag the English canonical docs.
- No automated cross-repository compatibility test runs in CI; the
  combined-compile check is a documented manual procedure.

## Current default feature set

```
default = ["plugin-samurai", "mcp-extension"]
```

Optional:

```
widget-blocks
effects
plugin-wasm
plugin-external
```

A build with `--no-default-features` contains the pure kernel without the
samurai plugin, MCP extension, blocks pack, effects or runtime hosts.

## Data and platform coverage

| Signal | Linux | macOS | Windows | Fallback |
|---|---|---|---|---|
| CPU usage and frequency | yes | yes | yes | yes |
| Per-core temperature | coretemp when sensor mapping is unambiguous | no per-core source; aggregate only | no per-core source; aggregate via thermal zones | none |
| Memory and swap | yes | yes | yes | yes |
| Disks and mount options | yes | yes | yes | partial |
| Network interfaces and IPs | yes | yes | yes | partial |
| Process fields | yes | yes | yes | partial |
| Batteries | sysfs | pmset | GetSystemPowerStatus (aggregate) | empty |
| GPU | nvidia-smi plus sysfs for AMD/Intel | nvidia-smi only | nvidia-smi only | empty |
| Package power | Intel RAPL | none | none | none |

## Maintenance note

When any item in this file changes, update it together with
`repositories.md` and the affected per-repository documents. Do not let the
snapshot silently drift.
