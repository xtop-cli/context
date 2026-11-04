# Plugin architecture

`xtop-plugin-samurai` is the reference plugin. It implements the `Plugin`
contract, runs heuristics over the process table, exposes a JSON action API
and renders one widget. It never depends on the kernel.

The canonical plugin id and action names are exported constants
(`PLUGIN_ID`, `actions::*`). The MCP extension consumes them at compile time,
so a rename breaks the build instead of breaking clients at runtime.

## Lifecycle

| Hook | Behavior |
|---|---|
| `manifest()` | id `samurai`, name `Samurai`, version from the crate, capabilities `ReadSystemInfo`, `KillProcesses`, `ModifyConfig`, `RenderWidgets` |
| `on_enable()` | marks the plugin enabled |
| `on_disable()` | marks the plugin disabled |
| `on_tick()` | counts ticks; every fifth tick reruns the analysis and replaces the alert list |
| `execute(action, params)` | JSON action API; records the last action and result |

The plugin starts enabled and reports itself through `plugin.status`.

## Action API

`execute` returns a JSON string or a `PluginError`. Params are plain strings
parsed per action; there is no general escaping scheme.

Actions:

- `system.summary`
- `processes.top`
- `processes.search`
- `process.info`
- `process.kill`
- `process.alerts`
- `threshold.set`
- `threshold.get`
- `config.get`
- `config.set`
- `alerts.status`
- `plugin.status`

See `samurai.md` for params and response details.

## Heuristic analysis

- Runs every fifth tick, which is roughly every five seconds at the default
  kernel cadence.
- Ten rules, at most 50 alerts per run.
- Reads the process table through the capability-gated snapshot.
- Keeps spawn-storm history across runs inside the plugin instance.
- Alerts are replaced wholesale by each completed run.

See `rules.md` for thresholds and pattern lists.

## Widget

The plugin registers one widget named `samurai` (the same plugin id). It
renders a bordered panel with the Samurai title and two lines of agent
status.

For the widget to appear:

1. the kernel is built with the plugin enabled;
2. the manifest declares `RenderWidgets`;
3. a layout references a widget named `samurai`.

Plugin widgets render ahead of pack widgets.

## Relationship to MCP

The MCP server lives in the `extensions` repository, not here.

- It depends on this crate at compile time.
- It targets `PLUGIN_ID` for every `execute_plugin` call.
- It builds its 12-tool table from the action constants.
- It translates typed MCP arguments into the string params syntax.

## Runtime host crates

- `xtop-wasm-contract`: serializable `Manifest`, `State` and `DrawList`,
  plus the draw operations. ABI version `"1"`.
- `xtop-widget-replay`: replays a draw list onto a ratatui frame.
- `xtop-wasm-guest`: Rust guest SDK and the `export_widget!` macro.
- `xtop-plugin-wasm`: loads modules, enforces fuel and memory limits, hot
  reloads on mtime change, caches the last good draw list.
- `xtop-plugin-external`: spawns helper processes, exchanges JSON lines,
  enforces per-response timeouts, never hot reloads.

See `runtime-widgets.md`, `wasm-widgets.md` and `external-widgets.md`.
