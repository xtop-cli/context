# Runtime widgets

Runtime widgets are widgets that are not compiled into the kernel. Two
optional hosts implement the same model: a shared state and draw-list
contract, plugin-path registration and tick-time rendering.

## Shared model

- The contract crate `xtop-wasm-contract` defines serializable `Manifest`,
  `State` and `DrawList` types.
- A guest answers a manifest request once and a render request once per
  tick.
- The host caches the returned draw list and replays it at frame time.
  Frames never call the guest, so a slow guest cannot stall a frame.
- Runtime widgets register through the plugin widget path.
- They take precedence over every compiled-in pack and can replace any
  widget name, including built-ins such as `cpu`.
- Layouts reference widgets by the manifest `name` (or the descriptor name
  for external widgets).

## Discovery

| Host | Directory | Override |
|---|---|---|
| WASM | `<config>/wasm/` | `XTOP_WASM_DIR` |
| External | `<config>/external/` | `XTOP_EXTERNAL_DIR` |

Config directories:

- Linux: `$XDG_CONFIG_HOME/xtop` or `~/.config/xtop`;
- macOS: `~/Library/Application Support/xtop`;
- Windows: `%APPDATA%\xtop`.

Duplicate names are resolved by sorted path order: the first one wins and
the duplicate is logged.

## Tick versus render

Tick:

- the host builds a `State`;
- calls the guest once;
- caches the draw list.

Render:

- replays the cached list;
- records the current widget area size.

A failed render leaves the last good list on screen. Both hosts log a
failure once per distinct message.

## Process cap

The manifest field `max_processes` caps `snapshot.processes`:

- default 50;
- clamped to 1 through 4096;
- processes are sorted by CPU descending before truncation.

Set it to 1 for widgets that never read processes, to keep payloads small.

## State payload

The state contains:

- `tick`, `unix_time`;
- last rendered `width` and `height`, zero before the first frame;
- `config` with theme, layout, interval and hostname;
- `alerts` with the three thresholds;
- `snapshot` with CPUs, memory, swap, disks, networks, processes, load,
  uptime, CPU temperature, disk I/O, batteries, GPUs and system info.

## Draw list

Operations:

- `block` with rect, border, title, fg, bg;
- `text` with rect, spans, alignment and wrapping;
- `gauge` and `bar` with rect, ratio, label and colors;
- `sparkline` with rect and data;
- `chart` with rect, datasets, bounds, marker and colors.

Coordinates are relative to the widget area and clipped by the host.

## Security

WASM host:

- 100 million fuel per host-to-guest call sequence;
- 64 MiB linear memory cap;
- only the `host.log` import is linked;
- rect clipping;
- guests cannot touch kernel types, ratatui or the terminal.

External host:

- the process runs with user privileges;
- the process boundary is the only isolation;
- each response is bounded by `timeout_ms`;
- the host is explicit that this is not a sandbox.

## Enabling

```
cargo build --features plugin-wasm
cargo build --features plugin-external
cargo build --features plugin-wasm,plugin-external
```

The default kernel build does not include either host. Without the feature,
the discovery directories are ignored.

## Demo assets

The layouts repository ships example layouts under `layouts/custom/` that
arrange the example widgets. The plugins repository ships the widgets
themselves under `examples/`. The kernel documents a demo flow that installs
examples into a temporary directory and launches xtop against it.
