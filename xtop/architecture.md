# Kernel architecture

The kernel is a single binary crate. `src/main.rs` is a thin dispatcher; the
application is organized into seven top-level areas plus the entry point.

## Module map

```
src/
  main.rs              CLI dispatch and usage
  commands/
    mod.rs
    run.rs             TUI run loop
    mcp.rs             MCP server command
    layout.rs          layout check and install
    theme.rs           --ct theme switching
    share/
      assets.rs        asset seeding and config persistence
      bootstrap.rs     startup wiring and plugin registration
      mod.rs
    plugins/
      mod.rs
      list.rs
      install.rs
      scaffold.rs
    widget/
      mod.rs
      list.rs
      install.rs
      scaffold.rs
  config/
    mod.rs
    io.rs              load and save
    keybinding.rs      keybinding schema
    schema.rs          persisted config
    platform/
      mod.rs
      linux.rs
      macos.rs
      windows.rs
      other.rs
      shared/
  plugins/
    mod.rs
    manager.rs         lifecycle, tick, key, collect, execute
    host.rs            HostState implementation
    extension_host.rs  ExtensionHost implementation and error mapping
  providers/
    mod.rs
    composite.rs       primary provider plus plugin extras
    sysinfo/
      mod.rs
      provider.rs      SysinfoProvider
      platform/
        mod.rs
        fallback.rs
        shared/gpu.rs
        linux/         battery, governor, gpu, interfaces, mounts, power,
                       temps, threads
        macos/         battery, interfaces, mounts, threads, users
        windows/       battery, interfaces, mounts, threads, users
  state/
    mod.rs
    app.rs             AppState
    app_tests.rs
    history.rs         bounded history buffers
    proc_history.rs    per-process CPU history
    palette.rs         command palette state
    users.rs           uid-to-name map
    view.rs            view state
    widget_state.rs    WidgetState implementation
  theme/
    mod.rs
    model.rs           Theme and role accessors
    loader.rs          JSONC loading and seeding paths
    contrast.rs        WCAG normalization
  ui/
    mod.rs
    screen.rs          top-level screen composition
    terminal.rs        terminal setup and teardown
    layout/
      mod.rs
      engine.rs        layout tree to ratatui, widget resolution
      pack_table.rs    compile-time pack catalog
    overlay/
      mod.rs
      help/mod.rs      help overlay
      palette/mod.rs   command palette overlay
    effects/
      mod.rs
      off.rs           no-op effect
      on.rs            active effect driver
```

## Areas and responsibilities

| Area | Responsibility |
|---|---|
| `commands` | CLI subcommands, startup, run loop, asset seeding |
| `config` | persisted schema, keybindings, platform config directories |
| `plugins` | plugin lifecycle, host views, extension host |
| `providers` | system sampling, platform probes, composite provider |
| `state` | runtime state, histories, process view, widget state |
| `theme` | theme model, loader, contrast normalization |
| `ui` | terminal, screen, layout engine, overlays, effects |

## Data flow

1. `main` dispatches a subcommand or starts the TUI.
2. `ensure_default_assets()` seeds themes and layouts when needed.
3. `commands::run` builds `AppState`, the provider, the plugin manager and
   the extension host.
4. On every tick:
   - the provider refreshes and produces one `SystemSnapshot`;
   - `AppState` caches it and pushes history points;
   - the plugin manager ticks plugins;
   - the UI renders the active layout from the cached sample.
5. On quit, `share::assets::save_config` persists runtime-owned fields.

The render path never resamples the system. Widgets and plugins in a frame
see the same sample.

## Render pipeline

1. `ui/screen.rs` handles terminal-too-small detection and chooses the
   effective layout.
2. The active `LayoutDef` is rendered by `ui/layout/engine.rs`.
3. The engine walks the layout tree, converts size constraints to ratatui
   constraints and splits rectangles.
4. For each widget leaf, `render_named` resolves the renderer:
   - plugin widget first;
   - chosen pack for the name;
   - base pack.
5. Per-instance layout `options` are set on `AppState` for the duration of
   the render call and exposed through `WidgetState::widget_options`.
6. Overlays (help, search, command palette) draw above the layout.
7. If the `effects` feature is enabled and the config selects an effect, the
   effect rewrites the buffer before flush.
8. Unknown widget names produce one stderr warning per name per process.

## Plugin hosting

- `PluginManager` registers plugins, creates per-plugin data directories,
  ticks them, dispatches keys, collects providers and widgets, executes
  named actions and disables them on shutdown.
- Errors are isolated per plugin and logged to stderr.
- `plugins/host.rs` implements `HostState` for `AppState`, so plugins see
  only the contract view.
- Provider collection is filtered by `ReadSystemInfo`; widget collection is
  filtered by `RenderWidgets`.
- `plugins/extension_host.rs` implements `ExtensionHost` and maps
  `PluginError` to `ExtensionError`.

## Provider composition

`CompositeProvider` wraps the primary provider.

- `refresh_all` refreshes the primary and every extra.
- `snapshot` delegates to the primary.
- `disk_io`, `batteries`, `gpu_info` and `system_info` use the primary when
  non-empty, otherwise the first non-empty extra.
- `kill_process` tries the primary, then the extras.

## Platform separation

All `cfg(target_os)` usage lives under `providers/sysinfo/platform/`. The
audit script fails if it appears anywhere else. Fallback platforms return
empty values rather than fabricated data.

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
- the uid-to-name map.

## Theme handling

`Theme` stores the explicit background and foreground plus a 16-entry
palette. `theme::loader` reads JSONC theme files from the platform config
directory; `theme::contrast` normalizes role colors against the background
at load time, never rewriting user files.

## Constraints enforced by the audit

`scripts/audit.sh` enforces:

- no `cfg(target_os)` outside `platform/` trees;
- no source file above 600 lines (warn above 300);
- no TODO/FIXME/XXX/HACK markers;
- at most 30 wildcard `pub use ...::*` re-exports;
- at most 2400 lines of code per top-level area;
- the dead pre-monocrate plugin tree is absent;
- exactly 12 embedded theme seeds including `miami`.

Module dependency-cycle detection and unused-public-item detection are not
implemented.
