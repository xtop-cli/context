# Shipped layout files

Ten JSONC files ship in `xtop-layout/layouts/default/`. Only the primary
`Dashboard` layout is embedded in the crate; the other six mode layouts and
the three detail presets are catalog assets installed with
`xtop layout download <name>` or `xtop config download --all`.

There is no fixed slot order for external layouts. Layouts are resolved by
their display `"name"`; mode names still map to the built-in `LayoutMode`
values, while every other name keeps the previously active mode.

## Mode layouts

| File | Name | Structure | Widgets |
|---|---|---|---|
| `dashboard.jsonc` | `Dashboard` | header 3, then a horizontal split at 45 percent (cpu 50 percent plus a vertical memory/storage/network column), then processes 52 percent | header, cpu, memory, storage, network, processes |
| `vertical.jsonc` | `Vertical` | vertical stack: header 3, cpu 8, memory 8, storage 6, network 5, processes fill | header, cpu, memory, storage, network, processes |
| `horizontal.jsonc` | `Horizontal` | header 3, then a horizontal split of cpu/memory/storage/network at 25 percent each | header, cpu, memory, storage, network |
| `cpu_focus.jsonc` | `CPU Focus` | header 3, cpu 60 percent, processes fill | header, cpu, processes |
| `memory_focus.jsonc` | `Memory Focus` | header 3, memory 60 percent, processes fill | header, memory, processes |
| `network_focus.jsonc` | `Network Focus` | header 3, horizontal split at 50 percent of network and disk_io, processes fill | header, network, disk_io, processes |
| `process_focus.jsonc` | `Process Focus` | header 3, horizontal split of 8 rows for cpu/memory/storage/network, processes fill | header, cpu, memory, storage, network, processes |

## Detail presets

| File | Name | Structure | Widgets |
|---|---|---|---|
| `detail_dashboard.jsonc` | `Detail Dashboard` | header 3; monitor band 58 percent with cpu fill and a 36 percent side column of summary 42 percent and sensors 58 percent; full-width processes 42 percent | header, cpu, summary, sensors, processes |
| `detail_network.jsonc` | `Detail Network` | header 3; network band 70 percent with network 60 percent and a 40 percent side column of summary 30 percent, disk_io 40 percent, memory 30 percent; processes 30 percent | header, network, disk_io, summary, memory, processes |
| `detail_processes.jsonc` | `Detail Processes` | header 3; an 8-row horizontal stat strip of summary 20 percent, cpu 28 percent, memory 18 percent, storage 17 percent, network 17 percent; processes fill | header, summary, cpu, memory, storage, network, processes |

## Options used by the presets

- `Detail Dashboard`: `cpu` with `cores` and `show_freq`; `processes` with `cpu: total`.
- `Detail Network`: `network` with `ifaces: all`; `processes` with `cpu: total`.
- `Detail Processes`: `processes` with `cpu: both`.

Options are display refinements only. Layouts remain valid without them and
renderers ignore unrecognized keys.

## Invariants

- Only `dashboard.jsonc` is embedded; every other file is a catalog asset.
- Downloaded and user layouts are resolved by name, not by a fixed slot.
- Every split must tile fully under the documented rules.
- `summary` and `sensors` are resolved at render time like every other name.
- None of the shipped layouts puts a size on the root.

## Tests that pin the contract

- only Dashboard is embedded;
- every shipped layout file parses;
- preset extras parse with widget options;
- detail presets tile every split fully;
- detail presets reference only registered widget names.
