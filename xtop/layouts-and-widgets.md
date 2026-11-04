# Layouts and widgets in the kernel

The kernel hosts the layout model from `xtop-layout` and the renderers from
widget packs. It owns the paths, the mode cycling and the render engine, but
not the layout format or the widget implementations.

## Layout modes

Seven built-in modes in fixed palette order:

1. `Dashboard`
2. `Vertical`
3. `Horizontal`
4. `CPU Focus`
5. `Memory Focus`
6. `Network Focus`
7. `Process Focus`

Three Detail presets are appended after the modes, not mapped to a mode:

- `Detail Dashboard`
- `Detail Network`
- `Detail Processes`

The first seven slots must never be renumbered. Overrides keep a slot; new
layouts append.

## Layout directories

| Platform | Directory |
|---|---|
| Linux | `$XDG_CONFIG_HOME/xtop/layouts` or `~/.config/xtop/layouts` |
| macOS | `~/Library/Application Support/xtop/layouts` |
| Windows | `%APPDATA%\xtop\layouts` |

Both `.jsonc` and `.json` are accepted. The directory is read at startup.
Restart the app after adding a file.

## Layout model

A layout is a recursive tree:

```
LayoutDef { name, root }
  root: Area (split or widget)
    split: { direction, size, areas }
    widget: { widget, size, options }
```

Size constraints:

- fixed integer: rows or columns;
- percentage string: share of the enclosing split;
- `"*"` or omitted: fill the remainder.

Direction:

- `vertical`: children stack top to bottom;
- `horizontal`: children sit left to right.

Tiling rules:

- pure percentage stacks should sum to 100;
- fixed lengths must sit next to a fill, not next to a percentage;
- a fill can absorb the remainder next to percentages that sum below 100.

## Widget names

The base pack registers:

`header`, `cpu`, `memory`, `storage`, `network`, `processes`, `disk_io`,
`battery`, `gpu`, `summary`, `sensors`.

- The default layouts reference all except `battery` and `gpu`.
- `battery` and `gpu` are reachable through fullscreen mode.
- Plugin and runtime widgets can register any name and take precedence.

## Per-instance options

A widget leaf may carry an `options` object. The layout crate stores it
verbatim; the kernel forwards it to the renderer through
`WidgetState::widget_options` for that render call.

Recognized keys by widget (see `widgets/widget-reference.md` for details):

- `cpu`: `chart`, `cores`, `show_freq`, `show_temp`.
- `processes`: `cpu`, `columns.memory`, `columns.user`, `columns.cmd`,
  `zebra`.
- `network`: `ifaces`.
- `storage`: `disks`.
- `memory`: `sections`.
- `disk_io`, `summary`, `sensors`, `header`, `battery`, `gpu`: glyph keys
  only.
- Every widget also accepts `charset` and `borders` as layout options, which
  beat the style configuration for that instance.

Unknown keys and malformed values are ignored.

## Pack selection

- `style.pack` selects a pack globally.
- `style.widgets.<name>.pack` selects a pack for one widget name.
- A missing name in the chosen pack falls back to the base pack.
- Plugin widgets precede every pack.

The kernel compiles packs into the binary. The base pack `xtop-widgets` is
always present. The blocks pack is behind the `widget-blocks` feature.

## Runtime widgets

Two opt-in features extend the widget surface without recompilation.

WASM host (`plugin-wasm`):

- discovers `.wasm` files in `<config>/wasm/`, or `XTOP_WASM_DIR`;
- uses wasmi with a 100 million fuel budget per call sequence, a 64 MiB
  memory cap and only the `host.log` import;
- hot-reloads on file mtime change;
- calls the guest once per tick and replays the cached draw list per frame.

External host (`plugin-external`):

- discovers descriptors in `<config>/external/`, or `XTOP_EXTERNAL_DIR`;
- spawns one helper process per descriptor;
- exchanges line-delimited JSON requests and responses;
- per-response timeout default 2000 ms, clamped to 100 through 60000 ms;
- no hot reload;
- runs with user privileges and is explicitly not sandboxed.

Both hosts register through the plugin widget path, keep precedence over
packs, and are referenced in layouts by their manifest name.

## Fullscreen and minimal views

- `f` toggles fullscreen for the focused widget.
- `F` cycles through CPU, memory, storage, network, processes, disk I/O, GPU,
  battery, then exits.
- Fullscreen looks up the widget's layout options from the current layout by
  name; if absent, it uses defaults.
- Below 60 columns or 14 rows the kernel uses a minimal renderer with CPU,
  memory and processes.
- Below 40x8 the screen shows a terminal-too-small notice.

## Responsive thresholds

| Mode and size | Effective layout |
|---|---|
| any, width under 60 or height under 14 | minimal |
| Dashboard, width under 80 | Vertical |
| Dashboard, width 80-99 or height under 28 | Compact |
| Dashboard, width 100+ and height 28+ | Dashboard |
| other modes at 60x14 or larger | requested mode |

## Layout validation and installation

- `xtop layout check <file>` validates structure with the same parser used
  at runtime.
- `xtop layout install <name>` fetches a community layout from the layouts
  repository and copies it into the user directory.
- Unknown widget ids are not a parse error. They render an empty area and
  warn once per name.
