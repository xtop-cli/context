# Samurai action reference

`xtop-plugin-samurai` exposes twelve actions through
`Plugin::execute(ctx, action, params)`. The return value is a JSON string.
Params use a hand-parsed mini-syntax.

## Actions

| Action | Params | Capability | Notes |
|---|---|---|---|
| `system.summary` | none | `ReadSystemInfo` | CPU average, memory GB and percent, process/disk/interface counts, uptime, hostname, alert count |
| `processes.top` | `count` or `count,filter=<regex>` | `ReadSystemInfo` | count defaults to 10 when unparsable; 0 is an error; filter matches name, cmd or exe |
| `processes.search` | `pattern` or `pattern,fields=a,b` | `ReadSystemInfo` | regex matched against any listed field; fields: name, cmd, user, state, exe, cwd; default field name; sorted by CPU descending; capped at 100 |
| `process.info` | `pid` | `ReadSystemInfo` | full process JSON; error when the pid is absent |
| `process.kill` | `pid` | `KillProcesses` | terminate a process |
| `process.alerts` | none | ungated read | full alert array from the last analysis run |
| `threshold.set` | `cpu,mem,disk` | `ModifyConfig` | three comma-separated percentages |
| `threshold.get` | none | ungated read | current thresholds |
| `config.get` | none | ungated read | theme, layout, interval, hostname |
| `config.set` | `interval_ms=<ms>`, `theme=<name>` or `layout=<name>` | `ModifyConfig` | one key per call; anything else is a parse error |
| `alerts.status` | none | ungated read | total and per-severity counts plus the top five alerts |
| `plugin.status` | none | ungated read | enabled flag, tick count, last action and result, active and critical alert counts |

## Params mini-syntax

- comma separates positional arguments;
- `key=value` selects a named argument for actions that accept one;
- no quoting or escaping layer exists;
- malformed params produce `PluginError::Recoverable`;
- unknown actions produce `PluginError::UnknownAction`.

## Process JSON fields

Process entries carry:

`pid`, `name`, `cpu`, `mem_bytes`, `state`, `user`, `cmd`, `exe`, `ppid`,
`threads`, `run_time`, `cwd`.

## Analysis schedule

- `on_tick` increments a counter.
- Every fifth tick runs the analysis and replaces the alert list.
- Errors propagate as `PluginError` and are logged by the kernel plugin
  manager.
- At most 50 alerts are kept per run.

## Example calls

```
system.summary
processes.top 5,filter=sshd
processes.search .*python.*,fields=name,cmd
process.info 4242
process.kill 4242
threshold.set 90,85,80
config.set theme=tokio
alerts.status
plugin.status
```

## MCP mapping

The MCP extension derives tool names by replacing `.` with `_`:

`system_summary`, `processes_top`, `processes_search`, `process_info`,
`process_kill`, `process_alerts`, `threshold_set`, `threshold_get`,
`config_get`, `config_set`, `alerts_status`, `plugin_status`.

Typed MCP arguments are translated into the mini-syntax before dispatch.
