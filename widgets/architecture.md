# Widget architecture

A widget is a crate. The crate is the designable and installable unit. Each
widget crate exposes one function:

```rust
pub fn render(f: &mut Frame, state: &dyn WidgetState, area: Rect)
```

Widgets never import kernel types. Every value comes from `WidgetState`.

## Crate model

```
xtop-widget-core
  chart.rs      per-cell colored chart engine and one-row spark helpers
  options.rs    layout options parsers and CPU chart/core types
  util.rs       formatting, palette roles, gauge gradients, temperature
                ramp, glyph resolution, frame prologue, Painter
  testkit.rs    WidgetState double and offscreen terminal helpers
```

Every widget crate depends on:

- `xtop-widget-api` (contract);
- `xtop-plugin-api` (model types);
- `xtop-widget-core` (shared engine);
- `ratatui`.

The aggregator crate `xtop-widgets` depends on all widget crates and builds
the base registry:

```rust
pub fn registry() -> HashMap<&'static str, WidgetRenderer> {
    // name -> Arc<dyn Fn(&mut Frame, &dyn WidgetState, Rect)>
}
```

`xtop-widget-blocks` is a sibling monolithic crate that uses the same
`xtop-widget-core` engine and registers its own subset of names.

## Dependency rule

The shared engine exists so widgets do not re-implement:

- palette roles and formatting;
- option parsing;
- glyph resolution;
- chart rendering;
- the temperature ramp;
- frame prologue and painter.

Canonical glyph mapping stays in the contract:

```rust
use xtop_widget_api::glyph::{border_for, marker_for, to_color, ASCII_BORDER};
```

Packs must not copy these mappings.

## Registry and resolution

The kernel's render engine resolves a widget name in this order:

1. plugin widgets, including runtime widgets;
2. the pack chosen by `style.widgets.<name>.pack` or `style.pack`;
3. the base pack.

If the chosen pack has no renderer for the name, the base pack is used. If
no pack provides the name, the area is empty and the kernel warns once.

## WidgetState inputs

Renderers use:

- `snapshot()` for the per-tick sample;
- theme accessors for colors;
- resolved `charset(name)` and `borders(name)`;
- histories for charts;
- `process_view()`, selection, search and sort state for the process table;
- `uid_to_name()` and `process_cpu_history()` for process rows;
- `widget_options()` and `logical_core_count()` for per-instance display
  options.

`widget_options()` is only valid during the render call. Unknown keys must
be ignored.

## Defensive rendering

- Return early when `snapshot()` is `None`.
- Handle tiny and empty rectangles without panicking.
- Never wrap a logical row; truncate with `...` or drop optional columns.
- Never fabricate data. Missing values produce an honest empty state.

The smoke tests render every registered widget of both packs at 100x34,
100x30, 80x24, 60x20, 40x15 and 20x10 with empty and sampled state.

## Pack installation in the kernel

`xtop widget install` adds:

- an optional dependency;
- a `widget-<name>` Cargo feature;
- a pack-table row and registry arm in `src/ui/layout/pack_table.rs`.

The pack is not enabled by default. Enable the feature, rebuild and select
the pack through the style configuration.
