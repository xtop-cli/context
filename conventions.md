# Cross-repo conventions

Rules that apply to every repository in the `xtop-cli` organization. A change
that violates one of these rules is incorrect even if it compiles.

## Version policy

- Every crate reports version `0.1.0`. The ecosystem is deliberately kept at
  an early version while the contracts settle.
- Do not bump crate versions for feature work. A release process may change
  this policy later; it has not changed yet.
- `rust-version = "1.87"` is declared in every crate manifest (workspace
  level plus per-member inheritance where applicable). Rust edition is
  `2021`. The edition 2024 sweep is deferred.
- The website is `xtop-web` version `0.1.0`, private, and is not published to
  npm.

## Dependency policy

Shared, pinned choices:

| Dependency | Version | Used by |
|---|---|---|
| `ratatui` | `0.30.2` | kernel, contracts, widget packs, layouts rendering path, effect tests |
| `serde` | `1` (derive) | contracts, kernel config, runtime widget contract |
| `serde_json` | `1` | contracts, kernel, runtime widget contract |
| `crossterm` | `0.29` | kernel |
| `sysinfo` | `0.39` | kernel |
| `toml` | `0.8` | kernel config |
| `anyhow` | `1` | kernel CLI |
| `wasmi` | `2` | `plugins/xtop-plugin-wasm` |
| `windows-sys` | `0.61` | kernel Windows probes |
| `next` | `15.5.x` | web |
| `react` / `react-dom` | `19.1.x` | web |
| `marked` | `15.x` | web markdown rendering |
| `typescript` | `5.8.x` | web |

Explicitly forbidden in the Rust repositories: `tokio`, `clap`, `chrono`.
The kernel is synchronous and argument parsing is hand-written on purpose.
Do not introduce an async runtime, a CLI framework or a date/time crate
without an explicit decision recorded in `api` or `xtop` documentation.

New dependencies must be justified by a real consumer. Prefer the standard
library and existing workspace dependencies.

## Contract ownership (single-source rules)

The `api` repository is the only place where cross-repository types are
defined. The rule set is recorded as DR-1 through DR-7 in the api docs.

| Rule | Surface | Owner |
|---|---|---|
| DR-1 | Data model (`SystemSnapshot`, `CpuInfo`, `ProcessInfo`, `AlertThresholds`, ...) and plugin protocol (`Plugin`, `PluginContext`, `HostState`, `PluginError`, `SystemDataProvider`) | `api/crates/plugin-api` |
| DR-2 | Widget render contract (`WidgetState`, `WidgetRenderer`, `WidgetRegistration`, glyph enums and helpers) | `api/crates/widget-api` |
| DR-3 | Layout format; widget names are plain strings; no layout type in api | `layouts` |
| DR-4 | Extension protocol (`Extension`, `ExtensionHost`, `ExtensionContext`) | `api/crates/extension-api` |
| DR-5 | Effect protocol (`Effect`, `EffectManifest`) | `api/crates/effect-api` |
| DR-6 | Ecosystem constants (samurai plugin id and action names) at the producer | `plugins` |
| DR-7 | Dependency and toolchain policy | all repositories |

Consequences:

- No consumer redefines a contract type.
- `xtop_widget_api::WidgetRegistration` and
  `xtop_plugin_api::PluginWidget` are distinct types with distinct render
  views (`WidgetState` versus `HostState`). They must never share a name.
- The samurai plugin id and action names are exported by
  `xtop-plugin-samurai`; the MCP extension imports them instead of
  duplicating strings.

## Naming

- Crates: `xtop-<area>-<name>`, for example `xtop-widget-cpu`,
  `xtop-plugin-samurai`, `xtop-extension-mcp`, `xtop-effect-fade`.
- The kernel crate is simply `xtop`.
- Layout names are display strings, not file names. Mode layouts use
  `Dashboard`, `Vertical`, `Horizontal`, `CPU Focus`, `Memory Focus`,
  `Network Focus`, `Process Focus`. Detail presets use `Detail Dashboard`,
  `Detail Network`, `Detail Processes`.
- Widget names are lowercase strings used by layouts: `header`, `cpu`,
  `memory`, `storage`, `network`, `processes`, `disk_io`, `battery`, `gpu`,
  `summary`, `sensors`.
- Theme names are lowercase: `x`, `berlin`, `bogota`, `helsinki`,
  `lahabana`, `london`, `madrid`, `miami`, `oslo`, `paris`, `praha`,
  `tokio`.
- Doc files use lowercase kebab-case (`plugin-contract.md`,
  `mcp-protocol.md`).

## Theme palette contract

Themes use format v2: explicit `background` and `foreground` plus a
16-entry `palette`. Slots have fixed roles and must not be repurposed.

| Slot | Role |
|---|---|
| 0 | legacy background alias (`ROLE_BG`) |
| 1 | alert / high fills |
| 2 | good / normal fills |
| 3 | warn / mid gradient |
| 4 | read / RX |
| 5 | write / TX / GPU |
| 6 | accent |
| 7 | legacy foreground alias (`ROLE_FG`) |
| 8 | dim / separators |
| 9-15 | bright multi-series ramp |

At load the kernel normalizes contrast against the explicit background with
WCAG floors. Shipped files stay canonical; normalization happens in memory.
Renderers must read colors by role.

## Layout contract

- Layout files are JSONC objects with `name` and `root`.
- A node is either a split (`direction` plus `areas`) or a widget leaf
  (`widget` plus optional `options`).
- Sizes are a fixed integer, a percentage string, `"*"`, or omitted (fill).
- The first seven embedded layouts map to the seven modes in fixed order.
  Never renumber them. Extra layouts (the three `detail_*` presets and user
  layouts) are appended after the modes and addressed by name.
- Merging is by exact, case-sensitive `name`. A user file with a built-in
  name replaces that built-in in place and keeps its palette slot.
- Tiling rules: pure percentage stacks must sum to 100; fixed lengths must
  sit next to a fill; percentages next to fills must sum below 100.
- Widget ids are unvalidated strings resolved at render time. An unknown id
  renders an empty area and emits one stderr warning per name per process.

## Code style

- `cargo fmt` is mandatory. CI checks `cargo fmt --all -- --check`.
- `cargo clippy --workspace --all-targets -- -D warnings` (or the
  single-crate equivalent) must pass.
- Prefer small modules; the kernel audit fails a source file above 600
  lines and warns above 300.
- Keep `cfg(target_os)` confined to `src/providers/sysinfo/platform/` trees
  in the kernel. The audit enforces zero occurrences elsewhere.
- No `TODO`, `FIXME`, `XXX` or `HACK` markers in kernel source; the audit
  fails on them.
- Public documentation comments (`//!` for modules) are expected in the
  kernel and contract crates.
- Do not add comments that restate the obvious. Explain decisions, not
  mechanics.

## Git and CI

- Default branch is `main` for every repository.
- Commit messages are descriptive, imperative or noun-phrase style. The AUR
  packaging repository uses the conventional `upgpkg: <pkg> <version>-<rel>`
  form, but that repository is outside the eight upstream repositories.
- Consumer manifests carry floating git dependencies during this cycle:
  `{ git = "https://github.com/xtop-cli/<repo>" }` without `rev` or `tag`.
  Temporary path dependencies and `[patch]` sections are development-only
  and must be removed before pushing.
- Rust repositories use local CI as their primary gate:
  `./scripts/ci.sh [stage]`. CI is being added gradually; do not add a new
  workflow to a repository without an explicit decision.
- Intentional workflows at this snapshot: `xtop` has the roadmap-to-issues
  sync and a tag-triggered `release.yml`; `web` has the GitHub Pages
  deployment workflow. The other Rust repositories have no workflows.
- Never commit `Cargo.lock` in the library workspaces. The kernel and the
  web application may keep their lockfiles.

## Documentation

- English is the canonical documentation language. Spanish translations
  live only in `web/i18n/es/docs/`.
- The kernel's user docs under `xtop/docs/` use HTML inside Markdown in
  several files. Match the existing file's style when editing it.
- Contract and ecosystem docs are plain Markdown.
- Do not duplicate a whole upstream document in this context repository.
  Summarize, link and record decisions. The upstream doc remains canonical.
- No emojis. No personal contact data. No machine-specific absolute paths
  except the documented `~/repos/xtop-cli/` convention.

## Security and data honesty

- Every capability-gated read or write returns a `Result`; denied
  capabilities produce `Recoverable` errors.
- Plugin widgets require `RenderWidgets`; plugin data providers require
  `ReadSystemInfo`; mutations require `KillProcesses` or `ModifyConfig`.
- External widgets run with user privileges and are explicitly not a
  sandbox. WASM widgets are sandboxed by wasmi fuel, memory and import
  limits but still share the host process.
- Hardware data is never fabricated. Missing sensors, batteries or GPUs
  produce honest empty states.
- Never log secrets or environment contents. The process model carries an
  `environ` field for consumers, but documentation must not reproduce it.
