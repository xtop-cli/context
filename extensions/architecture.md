# Extension architecture

`xtop-extension-mcp` is a server-style extension built on
`xtop-extension-api`. It never talks to the kernel directly. The kernel
hosts it behind the `mcp-extension` feature and launches it from
`xtop mcp`.

## Trait usage

`McpExtension` implements `Extension`:

- `manifest()` declares id `mcp`, name `xtop MCP server`, version from the
  crate, and the single server id `mcp`.
- `run_server(server_id, ctx)` rejects any id other than `"mcp"` with
  `ExtensionError::Unknown` and otherwise runs the stdio loop.

While running, the extension uses only:

- `ctx.tick()` - advance the monitoring tick and tick hosted plugins; called
  before each tool execution;
- `ctx.execute_plugin(plugin_id, action, params)` - run a plugin action; all
  tool calls funnel through it.

## Structure

`src/lib.rs`:

- `run_mcp_server(ctx)` - reads one line at a time, skips blanks, writes and
  flushes the response;
- `handle_line(line, ctx)` - parses one JSON-RPC line and dispatches;
  malformed JSON becomes a recoverable error;
- `handle_initialize`, `handle_tools_list`, `handle_tools_call` - the three
  protocol handlers;
- `TOOL_ACTIONS`, `tool_name()`, `action_for_tool()` - the tool table
  derived from samurai constants.

`src/tests.rs`:

- unit tests of the JSON-RPC layer;
- a mock `ExtensionHost` records `execute_plugin` calls and returns canned
  results;
- `tick()` is a no-op in the mock.

## Why it depends on the plugins repository

The tools this server advertises are the samurai plugin's actions. Before
the constants rule, the plugin id and the twelve action strings were
duplicated by hand, so a rename broke MCP clients silently at runtime.

Depending on `xtop-plugin-samurai` makes the coupling compile-time:

- `PLUGIN_ID` drives every `execute_plugin` call;
- `actions::*` drives the tool list and the argument translation;
- a rename on either side fails the extension build.

Trade-off: the extension build pulls in the plugin crate, and with it
ratatui and the plugin contract. That is acceptable for a kernel-hosted
server extension.

## Relationship to the kernel

- The kernel creates the extension and calls `run_server("mcp", ctx)`.
- The kernel implements `ExtensionHost` for `AppState` in
  `src/plugins/extension_host.rs`.
- The kernel maps `PluginError` onto `ExtensionError`.

## Related documents

- `mcp-protocol.md` - wire protocol and tool mapping.
- `../plugins/samurai.md` - action reference.
- `../plugins/rules.md` - heuristic thresholds.
