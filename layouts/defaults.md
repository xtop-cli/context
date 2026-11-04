# Embedded default layouts

Ten layouts are embedded in `xtop-layout`. The first seven are mode-bound and
occupy fixed slots. The last three are detail presets appended after the
modes.

## Mode layouts

| Slot | File | Name | Structure | Widgets |
|---|---|---|---|---|
| 0 | `dashboard.jsonc` | `Dashboard` | header 3, then a horizontal split at 45 percent (cpu 50 percent plus a vertical memory/storage/network column), then processes 52 percent | header, cpu, memory, storage, network, processes |
| 1 | `vertical.jsonc` | `Vertical` | vertical stack: header 3, cpu 8, memory 8, storage 6, network 5, processes fill | header, cpu, memory, storage, network, processes |
| 2 | `horizontal.jsonc` | `Horizontal` | header 3, then a horizontal split of cpu/memory/storage/network at 25 percent each | header, cpu, memory, storage, network |
| 3 | `cpu_focus.jsonc` | `CPU Focus` | header 3, cpu 60 percent, processes fill | header, cpu, processes |
| 4 | `memory_focus.jsonc` | `Memory Focus` | header 3, memory 60 percent, processes fill | header, memory, processes |
| 5 | `network_focus.jsonc` | `Network Focus` | header 3, horizontal split at 50 percent of network and disk_io, processes fill | header, network, disk_io, processes |
| 6 | `process_focus.jsonc` | `Process Focus` | header 3, horizontal split of 8 rows for cpu/memory/storage/network, processes fill | header, cpu, memory, storage, network, processes |

## Detail presets

| Slot | File | Name | Structure | Widgets |
|---|---|---|---|---|
| 7 | `detail_dashboard.jsonc` | `Detail Dashboard` | header 3; monitor band 58 percent with cpu fill and a 36 percent side column of summary 42 percent and sensors 58 percent; full-width processes 42 percent | header, cpu, summary, sensors, processes |
| 8 | `detail_network.jsonc` | `Detail Network` | header 3; network band 70 percent with network 60 percent and a 40 percent side column of summary 30 percent, disk_io 40 percent, memory 30 percent; processes 30 percent | header, network, disk_io, summary, memory, processes |
| 9 | `detail_processes.jsonc` | `Detail Processes` | header 3; an 8-row horizontal stat strip of summary 20 percent, cpu 28 percent, memory 18 percent, storage 17 percent, network 17 percent; processes fill | header, summary, cpu, memory, storage, network, processes |

## Options used by the presets

- `Detail Dashboard`: `cpu` with `cores` and `show_freq`; `processes` with
  `cpu: total`.
- `Detail Network`: `network` with `ifaces: all`; `processes` with
  `cpu: total`.
- `Detail Processes`: `processes` with `cpu: both`.

These options are display refinements only. The layouts remain valid without
them and renderers that do not recognize a key ignore it.

## Invariants

- Slots 0-6 must never be renumbered.
- New embedded extras must be appended at the end.
- Every split must tile fully under the documented rules.
- `summary` and `sensors` are resolved at render time like every other name.
- None of the defaults puts a size on the root.

## Tests that pin the contract

- default layout count and order;
- preset extras parse with widget options;
- detail presets tile every split fully;
- detail presets reference only registered widget names.
