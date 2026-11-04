# External runtime widgets

`xtop-plugin-external` turns helper processes into plugin widgets. The host
spawns one process per descriptor and exchanges one JSON object per line
over stdin and stdout. The guest can be any program in any language with a
runtime.

The process runs with the user's permissions and is the sandbox. Use WASM
widgets when an in-process capability-limited guest is required.

## Enabling and discovery

```
cargo build --features plugin-external
```

At startup the host scans `<config>/external/` for `*.json` descriptors,
sorted by path. `XTOP_EXTERNAL_DIR` overrides the directory.

## Descriptor

| Field | Required | Default | Notes |
|---|---|---|---|
| `name` | no | file stem | widget name layouts use |
| `description` | no | empty | may be replaced by the manifest description |
| `command` | yes | - | argv array; absolute paths recommended |
| `timeout_ms` | no | 2000 | per-response timeout, clamped 100-60000 |
| `max_processes` | no | 50 | clamped 1-4096; manifest wins when the descriptor uses the default |

Example:

```json
{
  "name": "lua-clock",
  "description": "UTC clock in Lua",
  "command": ["lua5.4", "/opt/xtop-widgets/lua-clock/widget.lua"],
  "timeout_ms": 2000,
  "max_processes": 1
}
```

A descriptor with invalid JSON or an empty command is skipped with a log
line. A process that fails to spawn or does not answer the initial manifest
request is skipped.

## Protocol

Host to guest:

| Line | Meaning |
|---|---|
| `{"type":"manifest"}` | ask for the manifest once at load |
| `{"type":"render","state":{...}}` | ask for a draw list once per tick |
| `{"type":"shutdown"}` | ask the process to exit |

Guest to host:

| Line | Meaning |
|---|---|
| `{"type":"manifest","manifest":{...}}` | manifest response |
| `{"type":"draw","ops":[...]}` | draw-list response |
| `{"type":"log","message":"..."}` | diagnostic, printed to stderr as `[external:<name>]` |

Rules:

- answer each request with exactly one non-log response;
- log lines may be sent at any time and are skipped while waiting;
- blank lines are ignored;
- never write non-JSON to stdout; diagnostics belong on stderr;
- on shutdown or stdin EOF, exit cleanly with status 0;
- the host kills and reaps the child after sending shutdown;
- a malformed response fails the current request and keeps the last cached
  draw list.

The state payload and draw operations are identical to the WASM contract.

## Lifecycle

| Phase | Behavior |
|---|---|
| Load | spawn, send `manifest`, wait up to `timeout_ms` |
| Tick | build state, send `render`, skip log lines, wait for draw or timeout, cache |
| Render | replay the cached list; the process is untouched |
| Shutdown | send shutdown, then kill and reap |

There is no hot reload. Descriptors and scripts are read once at startup.

## Failures

- a timeout is logged once per distinct message;
- on any tick failure the last valid draw list stays on screen and the next
  tick retries;
- a process that exits is not respawned;
- `max_processes` caps processes after a CPU-descending sort.

## Debug actions

| Action | Returns |
|---|---|
| `status` | name, path, command, ticks, ops, timeout, last error |
| `render` | cached draw list as JSON |
| `reload` | error: restart xtop to reload |

## Examples

- `examples/external/lua-clock/` - Lua 5.4 / LuaJIT.
- `examples/external/lua-histogram/` - Lua histogram and statistics.
- `examples/external/python-cpu/` - Python stdlib CPU widget.
- `examples/external/python-cpu-chart/` - moving average, standard
  deviation and percentiles.
- `examples/external/python-mem-regression/` - least-squares regression.
- `examples/external/node-clock/` - Node.js stdlib clock.
