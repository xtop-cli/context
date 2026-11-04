# Plugins and extensions in the kernel

The kernel hosts two kinds of optional behavior:

- plugins, the unit of domain behavior, integrated at compile time;
- extensions, server-style integrations that drive the kernel through
  plugins.

A plain build needs neither.

## Plugin architecture

Four kernel components:

| Component | Location | Purpose |
|---|---|---|
| contract types | `xtop-plugin-api` | trait, manifest, context, capabilities, model |
| `PluginManager` | `src/plugins/manager.rs` | registration, tick and key dispatch, execution, shutdown |
| `HostState` impl | `src/plugins/host.rs` | live-state view plugins may touch |
| `CompositeProvider` | `src/providers/composite.rs` | merge plugin data providers with the kernel provider |

Plugins never depend on kernel types. They see state only through
`PluginContext`, which wraps `HostState`.

## Manager behavior

- `register` reads the manifest id, creates the plugin data directory,
  builds a context with the declared capabilities and calls `on_enable`.
- `tick_all` calls `on_tick` on each plugin and logs errors per plugin.
- `handle_key` dispatches a key and returns true if a plugin consumed it.
- `collect_data_providers` only includes plugins with `ReadSystemInfo`.
- `collect_widgets` only includes plugins with `RenderWidgets`.
- `execute` finds a plugin by id and calls its action API.
- `disable_all` calls `on_disable` at shutdown.

A failing plugin is logged and does not crash the kernel.

## Capabilities

- `ReadSystemInfo`: snapshot, system info, top processes.
- `KillProcesses`: process termination.
- `ModifyConfig`: theme, layout, thresholds, interval.
- `RenderWidgets`: custom widget registration.
- `Custom(String)`: escape hatch with no built-in enforcement.

Denied capabilities produce `PluginError::Recoverable` with a message that
names the missing capability.

## Kernel wiring

Default features:

```
default = ["plugin-samurai", "mcp-extension"]
```

- `plugin-samurai` pulls `xtop-plugin-samurai`.
- `mcp-extension` pulls `xtop-extension-mcp`.

Optional:

- `widget-blocks` pulls the blocks widget pack.
- `effects` pulls the fade effect and the effect contract.
- `plugin-wasm` pulls the WASM runtime host.
- `plugin-external` pulls the external process host.

Registration happens in `src/commands/share/bootstrap.rs` under feature
guards.

## Plugin install flow

`xtop plugin install` is a self-modifying source workflow:

1. resolve a repository;
2. sparse-clone it into a temporary directory;
3. locate an `xtop-plugin-*` crate;
4. add an optional git dependency and a feature flag to the kernel
   `Cargo.toml`;
5. run `cargo check`;
6. clean up.

The plugin is registered but not enabled. Enable the feature and rebuild.

## Extension architecture

Extensions implement `xtop-extension-api` and run servers. The kernel hosts
them behind features and launches them from subcommands.

The only current extension is the MCP server:

- `xtop mcp` constructs `McpExtension` and calls `run_server("mcp", ctx)`;
- the server reads JSON-RPC 2.0 lines from stdin and writes responses to
  stdout;
- every tool call maps to `ctx.execute_plugin("samurai", action, params)`;
- `ctx.tick()` refreshes the kernel state before execution.

Extensions do not implement domain behavior themselves; they delegate to
plugins.

## MCP tool surface

Twelve tools derived from the samurai action constants:

`system_summary`, `processes_top`, `processes_search`, `process_info`,
`process_kill`, `threshold_set`, `threshold_get`, `config_get`,
`config_set`, `process_alerts`, `alerts_status`, `plugin_status`.

Tool name equals the action with `.` replaced by `_`. Typed MCP arguments
are translated to the samurai string params syntax.

## Runtime widget hosts

- `plugin-wasm` loads sandboxed modules through wasmi.
- `plugin-external` spawns helper processes and speaks line-delimited JSON.

Both register through the plugin widget path, so they take precedence over
compiled-in packs. Discovery directories are documented in
`plugins/runtime-widgets.md`.

## Manual plugin wiring

1. Scaffold or write the crate.
2. Publish it as a git repository.
3. Install it with `xtop plugin install <url>` or edit the manifest manually.
4. Register it in `bootstrap.rs` under its feature guard.
5. Enable the feature and rebuild.
6. Document its actions and capabilities.

## Security notes

- Capability checks are mandatory; do not bypass `PluginContext`.
- External widgets are not sandboxed.
- WASM widgets are constrained but share the host process.
- Process termination is limited to processes owned by the current user.
