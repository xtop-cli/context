# MCP protocol

`xtop-extension-mcp` implements a minimal Model Context Protocol server over
JSON-RPC 2.0, line-delimited on stdin and stdout. One JSON object per line,
one response per request, no HTTP.

The kernel launches it as `xtop mcp`.

## Transport framing

- every input line is trimmed; empty lines are skipped;
- malformed JSON terminates the server with a recoverable error and emits no
  response;
- every well-formed request produces exactly one response with
  `"jsonrpc": "2.0"` and the request id echoed.

## Handled methods

| Method | Purpose |
|---|---|
| `initialize` | announces protocol version `2024-11-05`, `capabilities.tools` and `serverInfo` |
| `tools/list` | the 12 tools generated from samurai action constants |
| `tools/call` | executes one tool against the hosted plugin |
| anything else | `-32601 Method not found: <method>` |

Not implemented: `notifications/initialized`, `ping`, `resources/*`,
`logging`, `instructions`, batching.

## Tool table

Tool name equals the action with `.` replaced by `_`. The table is generated
from `xtop-plugin-samurai`, so there is no second hand-maintained string
list.

| Tool | Action | Arguments to params |
|---|---|---|
| `system_summary` | `system.summary` | none |
| `processes_top` | `processes.top` | `count` (default 10), `filter` becomes `count[,filter=<regex>]` |
| `processes_search` | `processes.search` | `pattern`, `fields` becomes `pattern[,fields=<fields>]` |
| `process_info` | `process.info` | `pid` becomes `"<pid>"`, required |
| `process_kill` | `process.kill` | `pid` becomes `"<pid>"`, required |
| `threshold_set` | `threshold.set` | `cpu`, `mem`, `disk` become `"<cpu>,<mem>,<disk>"`, all required |
| `threshold_get` | `threshold.get` | none |
| `config_get` | `config.get` | none |
| `config_set` | `config.set` | one of `interval_ms`, `theme`, `layout` becomes `key=value` |
| `process_alerts` | `process.alerts` | none |
| `alerts_status` | `alerts.status` | none |
| `plugin_status` | `plugin.status` | none |

Descriptions and input schemas advertised by `tools/list` are maintained in
the extension; the action strings, plugin id and tool-name mapping come from
the plugin crate.

## tools/call flow

1. Resolve the tool name back to its action constant. Unknown name returns
   `-32601`.
2. Translate typed arguments into the plugin string params syntax. Missing
   required arguments return `-32602` with the argument name.
3. Call `ctx.tick()`.
4. Call `ctx.execute_plugin(PLUGIN_ID, action, params)`.
5. Wrap a successful response as
   `{"content":[{"type":"text","text":"<plugin json>"}]}`.

## Error mapping

| JSON-RPC code | Meaning | Origin |
|---|---|---|
| `-32601` | method or tool not found | unknown method or tool name |
| `-32602` | invalid params | missing required argument |
| `-32000` | server or execution error | extension error from plugin execution |

## Example session

```
-> {"jsonrpc":"2.0","id":1,"method":"initialize","params":{}}
<- {"jsonrpc":"2.0","id":1,"result":{"protocolVersion":"2024-11-05","capabilities":{"tools":{}},"serverInfo":{"name":"xtop","version":"0.1.0"}}}

-> {"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"processes_search","arguments":{"pattern":"sshd","fields":"name,cmd"}}}
<- {"jsonrpc":"2.0","id":2,"result":{"content":[{"type":"text","text":"[...]"}]}}
```

## Client configuration

```json
{
  "mcpServers": {
    "xtop": {
      "command": "xtop",
      "args": ["mcp"]
    }
  }
}
```

The build must include both `mcp-extension` and `plugin-samurai`, which is
the default feature set.

## Interactive testing

```
echo '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"system_summary","arguments":{}}}' | xtop mcp
xtop mcp
```

## Failure notes

- malformed JSON stops the server without a response;
- a plugin that is not hosted or a denied capability produces `-32000`;
- the analyze-based tools return stale or empty data until the first analysis
  run completes (every fifth tick).
