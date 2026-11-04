# xtop - kernel

`xtop` is a cross-platform TUI system monitor written in Rust. It renders
with ratatui, samples with sysinfo and platform probes, and hosts a modular
ecosystem of widgets, layouts, plugins, extensions and effects.

The kernel is a single crate, not a workspace. It is a thin host: metrics,
state, layout rendering, theme handling, configuration and command dispatch
live here; renderers and layouts come from sibling repositories.

## Features

- CPU usage per core and per thread, frequency, governor and temperature.
- RAM and swap monitoring with history charts.
- Network RX/TX per interface with cumulative totals and rates.
- Storage usage per mount and disk I/O rates per device.
- Process list with live search, CPU/Memory/PID/Name sorting, direction
  markers, PID-anchored selection and process termination.
- Battery probes on Linux, macOS and Windows.
- GPU monitoring through nvidia-smi on any platform, plus Linux sysfs for
  AMD/Intel; honest empty states where no API exists.
- Twelve themes in JSONC with WCAG contrast normalization.
- Seven layout modes plus three Detail presets, custom layouts and
  fullscreen mode.
- Configurable alert thresholds and keybindings.
- Persistent configuration and automatic asset seeding.
- Plugin system with capability enforcement.
- MCP server for AI agents.
- CLI theme switching that running instances follow.
- Optional runtime widgets: sandboxed WASM modules or helper processes.

## Install

Linux:

```
curl -fsSL https://raw.githubusercontent.com/xtop-cli/xtop/main/install.sh | bash
```

Windows PowerShell (requires Rust):

```
irm https://raw.githubusercontent.com/xtop-cli/xtop/main/install.ps1 | iex
```

macOS and other platforms:

```
cargo install --git https://github.com/xtop-cli/xtop --all-features
```

From source:

```
git clone https://github.com/xtop-cli/xtop.git
cd xtop
cargo run --release
```

## Quick keys

| Key | Action |
|---|---|
| `q` | quit and save configuration |
| `?` | help overlay |
| `t` / `T` | next / previous theme |
| `l` | next layout |
| `f` / `F` | toggle / cycle fullscreen |
| `/` | process search |
| `ctrl+p` | command palette |
| `k` | kill selected process |
| `s` | cycle process sort column |

## Documentation map

- `architecture.md` - module map, data flow, render pipeline.
- `cli.md` - commands, flags and environment variables.
- `configuration.md` - config schema and persistence.
- `theming.md` - theme format, roles, contrast, shipped themes.
- `layouts-and-widgets.md` - modes, presets, packs and runtime widgets.
- `plugins-and-extensions.md` - hosting, capabilities and MCP.
- `providers.md` - data collection and platform probes.
- `operations.md` - build, test, audit, packaging, release.

Upstream user documentation lives in `xtop/docs/`:

- `features.md`, `installation.md`, `usage.md`, `configuration.md`,
  `customization.md`, `colors.md`, `plugin.md`, `multi-repo.md`,
  `design.md`.
- `ROADMAP.md`, `CHANGELOG.md`, `CONTRIBUTING.md`.

## Default and optional features

Default:

```
plugin-samurai
mcp-extension
```

Optional:

```
widget-blocks
effects
plugin-wasm
plugin-external
```

`cargo build --no-default-features` builds the pure kernel without samurai,
MCP, the blocks pack, effects or runtime widget hosts.

## License

MIT.
