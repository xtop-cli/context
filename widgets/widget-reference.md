# Widget reference

Both packs draw a standard frame: title, all borders, a border set resolved
from per-widget configuration through `border_for`, and theme foreground and
background.

## Global glyph options

Every widget accepts two layout options, with precedence
`layout-node option > style config > default`:

| Key | Values | Default |
|---|---|---|
| `charset` | `braille`, `dot`, `block`, `half_block`, `bar` | `braille` |
| `borders` | `native`, `rounded`, `double`, `plain`, `ascii` | `native` |

Malformed or unknown values are ignored and the configuration value applies.

## Palette roles

0 background, 1 alert, 2 good, 3 warn, 4 read/RX, 5 write/TX, 6 accent,
7 foreground, 8 dim, 9-15 multi-series ramp.

## Widgets

### header

One summary line when width is at least 80, otherwise two. Segments: host,
theme, layout, uptime, load averages colored by their share of logical
cores. Appends fullscreen and search markers when active.

Options: glyph keys only.

### cpu

One normalized row per core: label, percent, optional frequency, per-core
heat mark, optional temperature, gradient bar. Uses two columns when each
column keeps its minimum width.

Options:

| Key | Values | Default | Effect |
|---|---|---|---|
| `chart` | `average`, `per-core` | `average` | history plot mode |
| `cores` | `all` or `0,2,4-7` | `all` | restrict shown cores |
| `show_freq` | bool | `false` | append frequency |
| `show_temp` | bool or `auto` | `auto` | per-core temperature cell and heat mark |

Below the rows: a unified usage, temperature and power line when space
allows, then a history chart. Without chartable history it shows a numeric
summary. Temperatures and power are only drawn when real data exists.

### processes

Fixed columns: PID, Name, CPU spark, CPU%, Mem, User, Command, separated by
dim bars. The sorted column carries a direction marker.

Options:

| Key | Values | Default | Effect |
|---|---|---|---|
| `cpu` | `core`, `total`, `both` | `core` | CPU column basis |
| `columns.memory` | bool | `true` | show memory column |
| `columns.user` | bool | `true` | show user column |
| `columns.cmd` | bool | `true` | show command column |
| `zebra` | bool | `true` | alternating row background |

Columns drop right to left as width shrinks. User names resolve through
`uid_to_name`, falling back to the numeric uid. Command prefers the full
argv, then the single command, then the executable path, then `?`. The cpu
spark uses recent per-process samples and an honest placeholder when empty.
The selection is PID-anchored and the viewport keeps it visible.

### network

Per-interface rows with name, activity bar, RX/TX rates and cumulative
totals, degrading through width tiers. Aggregate lines appear below 26 inner
columns.

Options:

| Key | Values | Default | Effect |
|---|---|---|---|
| `ifaces` | `all` or a list | `all` | interfaces covered by rows and aggregate |

The machine-wide RX/TX rate chart uses fixed roles 4 and 5 and fills the
leftover rows.

### storage

One row per mount with label, percent, bar and, on wide rows, used and free
amounts plus a free-share braille bar. Tall boxes render three-line meter
blocks per disk.

Options:

| Key | Values | Default | Effect |
|---|---|---|---|
| `disks` | `all` or a list of mount points | `all` | mounts shown |

### memory

RAM, available and swap meter rows with used and free amounts on wide rows
and a RAM free-share spark. A RAM history chart fills leftover rows.

Options:

| Key | Values | Default | Effect |
|---|---|---|---|
| `sections` | list of `memory`, `available`, `swap` | all three | meter rows drawn |

### disk_io

One row per device with read and write rates and optional rate bars.
Aggregate read/write history chart fills leftover rows.

Options: glyph keys only.

### summary

Compact panel with load averages, CPU gauge, memory gauge, process counts by
state, and uptime. Leftover rows host the load history chart, or an inline
sparkline at small heights.

Options: glyph keys only.

### sensors

Per-core temperature grid colored by the temperature ramp; the title carries
the maximum. Without any temperature data it shows an honest line plus load
averages. Leftover rows host the load chart.

Options: glyph keys only.

### battery

One gauge per battery with name, percent, state and time to full or empty.
Shows `No battery data available` when empty.

Options: glyph keys only.

### gpu

One gauge per GPU with name, percent, memory used/total and temperature.
Shows `No GPU data available` when empty.

Options: glyph keys only.

`battery` and `gpu` are not referenced by default layouts; the kernel
reaches them through fullscreen mode.

## Chart engine

The shared engine renders colored history plots:

- braille charset: 4 sub-rows per text row;
- block and half_block: 8 sub-rows per text row;
- dot and bar: classic ratatui chart path;
- a height-1 plot uses the 8-level block ramp as a sparkline;
- one-row sparks are discrete and use braille or block glyphs by charset;
- fixed-role series use the series role color;
- heat series use the gauge gradient with the relevant alert threshold.

Minimum widths: cpu 12, memory 14, network 16, disk_io 16, summary/sensors
load chart 12. Narrower areas fall back to a numeric summary.

## Temperature ramp

- at or under 45 C: good role;
- at 60 C: warn role;
- at or over 80 C: alert role;
- intermediate colors are interpolated between role colors.

No new palette slot is introduced.

## Blocks pack

Registers `cpu`, `memory`, `processes`, `network`, `storage`, `disk_io`,
`summary` and `sensors`. It keeps an ASCII identity with `#` fills and
`|`-separated tables while using the same shared engine, glyph resolution
and option parsing. Chart glyphs still honor the resolved charset.

## Behavioral guarantees

- No widget mutates state or names kernel types.
- CPU history is per core and index-aligned with the snapshot cores.
- Network, disk and load histories are machine-wide aggregates.
- Missing data yields an honest fallback, never a fabricated reading.
