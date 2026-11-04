# Widget render contract

`xtop-widget-api` is the contract for widget packs. A pack registers
renderers by widget name; the kernel resolves the name at render time.

Three pieces:

- `state::WidgetState` - the read-only view a renderer receives;
- `glyph` - glyph enums and canonical mappings to ratatui primitives;
- `renderer::{WidgetRenderer, WidgetRegistration}` - renderer registration.

The crate depends on `ratatui`, `serde` and `xtop-plugin-api`. It never
depends on the kernel. The root re-exports `WidgetState`,
`WidgetRegistration`, `WidgetRenderer`, `ChartCharset` and
`WidgetBorders`; glyph helper functions are reachable only through
`xtop_widget_api::glyph`.

## WidgetState

Renderers draw against this trait, implemented by the kernel over its live
application state. It is read-only by design.

Methods grouped as in the source:

Sample:

- `snapshot() -> Option<&SystemSnapshot>` - current tick sample; `None`
  before the first tick.

Theme:

- `theme_name() -> &str`
- `theme_fg() -> &[u8; 3]`
- `theme_bg() -> &[u8; 3]`
- `theme_palette() -> &[[u8; 3]; 16]`
- `alerts() -> AlertThresholds`

Glyph style, already resolved:

- `charset(widget: &str) -> ChartCharset`
- `borders(widget: &str) -> WidgetBorders`

History:

- `cpu_history() -> &[VecDeque<(f64, f64)>]` - per core, `(x, y)`
- `mem_history() -> &VecDeque<(f64, f64)>`
- `net_rx_history() -> &VecDeque<(f64, f64)>` - summed rate in bytes/s
- `net_tx_history() -> &VecDeque<(f64, f64)>` - summed rate in bytes/s
- `disk_read_history()` - aggregate read rate, default empty
- `disk_write_history()` - aggregate write rate, default empty
- `load_history()` - 1-minute load average, default empty

View and control:

- `search_query() -> &str`
- `process_selected_pid() -> Option<u32>`
- `process_sort_label() -> &str`
- `process_sort_desc() -> bool` - default false
- `layout_name() -> &str`
- `is_searching() -> bool`
- `fullscreen_label() -> Option<&str>`
- `sys_info() -> SystemInfo` - owned copy
- `process_view() -> Vec<&ProcessInfo>` - filtered and sorted rows

Process mapping:

- `uid_to_name(uid) -> Option<String>` - default `None`
- `process_cpu_history(pid) -> Vec<f64>` - default empty, oldest first

Layout options:

- `logical_core_count() -> usize` - default 1
- `widget_options() -> Option<&serde_json::Value>` - default `None`; only
  valid during the render call; unknown keys ignored

Semantics:

- Glyph style arrives resolved; renderers do not read configuration.
- History entries are `(x, y)` pairs for ratatui charts.
- `process_view()` is the single per-tick sample filtered by search and
  sorted by the chosen column; selection is anchored by PID.
- `snapshot()` returns a reference; do not clone per frame.
- Methods added for layout options and process mapping have defaults, so
  existing implementors compile unchanged and output without options is
  byte-identical.

## Renderer registration

```rust
pub type WidgetRenderer =
    Arc<dyn Fn(&mut Frame, &dyn WidgetState, Rect) + Send + Sync>;

pub struct WidgetRegistration {
    pub name: String,
    pub render: WidgetRenderer,
}
```

A pack registers one renderer per widget name. `WidgetRegistration` is the
canonical type and lives only in this crate. Packs today return a
`HashMap<&'static str, WidgetRenderer>` from `registry()`; the kernel
resolves `(pack, name)` at render time.

## Glyph enums

Both derive `Clone, Copy, Debug, PartialEq, Eq, Serialize, Deserialize` and
use `#[serde(rename_all = "snake_case")]`.

| Enum | Variants | Default |
|---|---|---|
| `ChartCharset` | `Braille`, `Dot`, `Block`, `HalfBlock`, `Bar` | `Braille` |
| `WidgetBorders` | `Native`, `Rounded`, `Double`, `Plain`, `Ascii` | `Native` |

JSON values match the snake-case names: `"braille"`, `"half_block"`,
`"plain"`, and so on.

## Canonical glyph helpers

Import from the single canonical path:

```rust
use xtop_widget_api::glyph::{ASCII_BORDER, border_for, marker_for, to_color};
```

- `to_color([u8; 3]) -> Color` returns `Color::Rgb` verbatim.
- `border_for(WidgetBorders) -> Set<'static>`:

| Value | Border set |
|---|---|
| `Native` | ratatui `border::PLAIN` (single-line box drawing) |
| `Rounded` | `border::ROUNDED` |
| `Double` | `border::DOUBLE` |
| `Plain` | `ASCII_BORDER` |
| `Ascii` | `ASCII_BORDER` |

`ASCII_BORDER` uses `+` for corners, `|` for verticals and `-` for
horizontals.

- `marker_for(ChartCharset) -> Marker` maps directly to the ratatui marker
  of the same name.

Packs must not re-implement these mappings. A deliberate divergence must be
documented.

## Pack implementation

A pack returns a name-to-renderer registry:

```rust
use std::collections::HashMap;
use std::sync::Arc;
use ratatui::prelude::*;
use xtop_widget_api::{WidgetRenderer, WidgetState};

pub fn registry() -> HashMap<&'static str, WidgetRenderer> {
    let mut m: HashMap<&'static str, WidgetRenderer> = HashMap::new();
    m.insert("cpu", Arc::new(cpu::render));
    m
}
```

Rules:

- Render only from `WidgetState`; never name kernel types.
- Guard for `snapshot() == None` and for tiny or empty areas.
- Use the canonical glyph helpers.
- Honor the resolved charset and borders.

## Resolution order

The kernel render engine checks:

1. plugin widgets (including runtime widgets);
2. the user's chosen pack for the name;
3. the base pack.

An unknown name renders an empty area and warns once per process.

## Plugin widgets versus pack registrations

- Pack renderers see `&dyn WidgetState`.
- Plugin widgets see `&dyn HostState` and use `PluginWidget`.
- The two contracts are distinct and must not share a name.
