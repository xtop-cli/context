# Kernel architecture

The kernel is a single Cargo package with one library (`src/lib.rs`) and one
binary (`src/main.rs`). `main` only forwards to `xtop::cli::run()`. The
library is organized into three top-level areas with a strict dependency
direction:

```
core  <-  tui
  ^
  |
 cli  (the only cli -> tui edge is cli/commands/tui.rs)
```

## Module map

```
src/
  main.rs                  binary entry: xtop::cli::run()
  lib.rs                   library surface: pub mod cli/core/tui
  catalog.json             official downloadable asset catalog
  core/
    mod.rs
    assets/
      mod.rs               external asset facade
      base.rs              base x theme + Dashboard seeding, --gen-config
      catalog.rs           catalog schema, validation and lookup
      download.rs          curl/file fetch, SHA-256, atomic install
      installed.rs         local assets.json metadata
    bootstrap.rs           startup wiring: provider, themes, layouts, plugins
    config/
      mod.rs
      io.rs                load and save
      persist.rs           persisted theme/layout selection for the TUI
      keybinding.rs        keybinding schema
      schema.rs            persisted config
      platform/            Linux, macOS, Windows and fallback config dirs
    plugins/
      mod.rs
      manager.rs           lifecycle, tick, key, collect, execute
      host.rs              HostState implementation
      extension_host.rs    ExtensionHost implementation and error mapping
    providers/
      mod.rs
      composite.rs         primary provider plus plugin extras
      sysinfo/
        mod.rs
        provider.rs        SysinfoProvider
        platform/          Linux, macOS, Windows and fallback probes
    state/
      mod.rs
      app.rs               AppState
      app_tests.rs
      history.rs           bounded history buffers
      persist.rs           runtime state -> config.json
      proc_history.rs      per-process CPU history
      users.rs             uid-to-name map
      view.rs              shared view-control types (fullscreen/input/palette)
    theme/
      mod.rs
      model.rs             Theme and role accessors
      loader.rs            JSONC loading and themes directory
      contrast.rs          WCAG normalization
  cli/
    mod.rs                 argument dispatch and --version
    help/
      mod.rs               global help and shared helpers
      assets.rs            theme/layout/config help
      dev.rs               plugin/widget help
      system.rs            monitoring/diagnostics/completions help
    commands/
      mod.rs
      addon.rs             unified theme/layout addon management
      completions.rs       shell completion scripts
      config.rs            config download/generate/get/set/list/reset/path
      download.rs          shared download flag parsing/reporting
      layout.rs            layout list/set/download/check/path
      mcp.rs               MCP server command
      monitor.rs           shared headless sampling and rendering
      processes.rs         one-shot process list
      status.rs            one-shot system snapshot
      system.rs            info and doctor
      theme.rs             theme list/set/download/available/path
      tui.rs               the single cli -> tui launch edge
      update.rs            self-update with installation-medium detection
      watch.rs             periodic status snapshots
    dev/
      mod.rs               developer tooling boundary
      plugins/             list/install/scaffold
      widget/              list/install/scaffold
  tui/
    mod.rs
    run.rs                 TUI run loop
    input.rs               key mapping and mode dispatch
    actions.rs             keybinding action -> state mutations
    palette.rs             command bar state logic
    widget_state.rs        WidgetState implementation for AppState
    screen.rs              top-level composition
    terminal.rs            platform backend facade
    terminal_common.rs     shared crossterm/ratatui primitives
    unix/
      mod.rs
      terminal.rs          Unix terminal lifecycle
    windows/
      mod.rs
      terminal.rs          Windows terminal lifecycle
    fallback/
      mod.rs
      terminal.rs          non-Unix/non-Windows terminal lifecycle
    layout/
      mod.rs
      engine.rs            layout tree to ratatui
      pack_table.rs        compile-time pack catalog
    overlay/
      mod.rs
      help/mod.rs          help overlay
      palette/mod.rs       command bar overlay
    effects/
      mod.rs
      off.rs               no-op host
      on.rs                active effect driver
```

## Areas and responsibilities

| Area | Responsibility |
|---|---|
| `core` | configuration, providers, live state, themes, plugins and shared bootstrap; frontend-agnostic |
| `cli` | argument dispatch, layered help, headless monitoring, config/asset management and source-management tools; consumes `core` |
| `tui` | terminal lifecycle, input, screen composition, layout rendering, overlays and effects; consumes `core` |
| `tui/unix`, `tui/windows`, `tui/fallback` | platform terminal backends selected by `tui/terminal.rs` |

The audit enforces the direction: `core` never references `cli` or `tui`,
`tui` never references `cli`, and `cli` reaches the TUI only through
`src/cli/commands/tui.rs`.

## Data flow

1. `main` calls `cli::run`; the dispatcher selects a command or the TUI.
2. `--change-theme`, `mcp` and the TUI call
   `core::assets::ensure_default_assets()`, which seeds only `x` and
   Dashboard. `--gen-config` additionally writes `config.json`.
3. The TUI/cli bootstrap (`core::bootstrap::initialize_state`) builds
   `AppState`, the provider, themes, layouts and the plugin manager.
4. On every tick:
   - the provider refreshes and produces one `SystemSnapshot`;
   - `AppState` caches it and pushes history points;
   - the plugin manager ticks plugins;
   - the TUI renders the active layout from the cached sample.
5. On quit, `core::assets::save_config` persists runtime-owned fields.

The render path never resamples the system. Widgets and plugins in a frame
see the same sample.

## Render pipeline

1. `tui/screen.rs` handles terminal-too-small detection and chooses the
   effective layout.
2. The active `LayoutDef` is rendered by `tui/layout/engine.rs`.
3. The engine walks the layout tree, converts size constraints to ratatui
   constraints and splits rectangles.
4. For each widget leaf, `render_named` resolves the renderer:
   - plugin widget first;
   - chosen pack for the name;
   - base pack.
5. Per-instance layout `options` are set on `AppState` for the duration of
   the render call and exposed through `WidgetState::widget_options`.
6. Overlays (help, process filter, command bar) draw above the layout.
7. If the `effects` feature is enabled and the config selects an effect, the
   effect rewrites the buffer before flush.
8. Unknown widget names produce one stderr warning per name per process.

## Plugin hosting

- `PluginManager` registers plugins, creates per-plugin data directories,
  ticks them, dispatches keys, collects providers and widgets, executes
  named actions and disables them on shutdown.
- Errors are isolated per plugin and logged to stderr.
- `core/plugins/host.rs` implements `HostState` for `AppState`, so plugins
  see only the contract view.
- Provider collection is filtered by `ReadSystemInfo`; widget collection is
  filtered by `RenderWidgets`.
- `core/plugins/extension_host.rs` implements `ExtensionHost` and maps
  `PluginError` to `ExtensionError`.

## Provider composition

`CompositeProvider` wraps the primary provider.

- `refresh_all` refreshes the primary and every extra.
- `snapshot` delegates to the primary.
- `disk_io`, `batteries`, `gpu_info` and `system_info` use the primary when
  non-empty, otherwise the first non-empty extra.
- `kill_process` tries the primary, then the extras.

## Platform separation

All `cfg(target_os)` usage lives under `core/config/platform/` and
`core/providers/sysinfo/platform/`. The TUI selects its terminal backend
with `cfg(unix)`/`cfg(windows)` in `tui/terminal.rs`; the backend modules
under `tui/unix/` and `tui/windows/` own the platform entry points. The
audit fails if `cfg(target_os)` appears anywhere else. Fallback platforms
return empty values rather than fabricated data.

## State

`AppState` holds:

- the cached snapshot and last refresh timestamp;
- bounded histories: per-core CPU, memory, network RX/TX rates, disk
  read/write rates and load;
- per-process CPU history ring;
- the process selection, search query and sort state;
- theme, layout definitions, layout index and mode;
- alert thresholds, update interval and keybindings;
- widget style resolution;
- active widget options for the current render call;
- the uid-to-name map;
- shared view-control state consumed by the TUI (fullscreen, input mode,
  command bar).

TUI-only action dispatch and palette logic live under `tui/`, not in core.

## Theme handling

`Theme` stores the explicit background and foreground plus a 16-entry
palette. `core/theme/loader.rs` reads JSONC theme files from the platform
config directory; `core/theme/contrast.rs` normalizes role colors against
the background at load time, never rewriting user files.
`core/config/persist.rs` exposes the persisted theme and layout selection to
the CLI and the TUI run loop.

## Constraints enforced by the audit

`scripts/audit.sh` enforces:

- no `cfg(target_os)` outside `platform/` trees;
- no source file above 600 lines (warn above 300);
- no TODO/FIXME/XXX/HACK markers;
- at most 30 wildcard `pub use ...::*` re-exports;
- at most 2400 lines of code per area or `core` sub-area, with `platform/`
  budgeted separately;
- the frontend dependency direction (`core`/`tui`/`cli` rules above);
- the dead pre-monocrate plugin tree is absent;
- exactly one embedded theme seed (`x`) and the root `catalog.json`.

Module dependency-cycle detection and unused-public-item detection are not
implemented.
