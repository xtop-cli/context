# Authoring a widget crate

## 1. Choose the unit

A widget is a crate under `xtop-widget-<name>/`. Copy an existing crate or
run the kernel's `xtop widget scaffold <name>`, which emits the same shape.

## 2. Crate manifest

Keep the contract dependencies:

```toml
[dependencies]
ratatui.workspace = true
xtop-widget-api.workspace = true
xtop-plugin-api.workspace = true
xtop-widget-core = { path = "../xtop-widget-core" }

[dev-dependencies]
xtop-widget-core = { path = "../xtop-widget-core", features = ["testkit"] }
```

Version stays `0.1.0` and inherits workspace fields.

## 3. Implement render

```rust
use ratatui::prelude::*;
use xtop_widget_api::WidgetState;

pub fn render(f: &mut Frame, state: &dyn WidgetState, area: Rect) {
    let Some(snapshot) = state.snapshot() else { return };
    // draw inside area
}
```

Rules:

- Draw only inside `area`.
- Return early when there is no sample.
- Handle tiny rectangles without panicking.
- Do not wrap logical rows; truncate instead.
- Use `xtop-widget-core` for the frame, colors, charts and formatting.
- Use the canonical glyph helpers from `xtop_widget_api::glyph`.

## 4. Read state correctly

- `state.theme_fg()` and `state.theme_bg()` return `&[u8; 3]`.
- `state.theme_palette()[index]` returns `[u8; 3]`.
- `state.charset(name)` and `state.borders(name)` are already resolved.
- `state.widget_options()` is `None` when the layout node has no options.
- `state.logical_core_count()` normalizes per-core CPU values.
- `state.process_view()` is filtered and sorted by the kernel.
- `state.uid_to_name(uid)` and `state.process_cpu_history(pid)` are display
  helpers; fall back honestly when they return `None` or an empty series.

## 5. Register

Built-in crates are registered by the aggregator in `src/lib.rs`:

```rust
m.insert("mywidget", Arc::new(xtop_widget_mywidget::render));
```

Add the crate to:

- the workspace `members` list;
- the aggregator dependencies;
- the pack's registration table.

For a separate pack, register the name in its own registry and add it to the
kernel pack catalog with `xtop widget install`.

## 6. Test

Widget tests use the `testkit` double:

```rust
#[cfg(test)]
mod tests {
    use super::*;
    // build a state double, render into an offscreen terminal,
    // assert rows and bounds
}
```

Cover at minimum:

- empty state (no snapshot);
- a normal sampled state;
- small areas and wide areas;
- each recognized layout option;
- every pack-specific rendering branch.

Smoke tests should render at 100x34, 100x30, 80x24, 60x20, 40x15 and 20x10.

## 7. Run local CI

```
cd ~/repos/xtop-cli/widgets
./scripts/ci.sh
```

## 8. Install into a kernel

From a kernel checkout:

```
xtop widget scaffold mywidget
xtop widget install /path/to/xtop-widget-mywidget
```

`widget install` edits the kernel manifest and pack table and runs
`cargo check`. Enable the generated feature and rebuild.

## Checklist

- crate name follows `xtop-widget-<name>`;
- render signature matches the contract exactly;
- no kernel type is imported;
- canonical glyph helpers are used;
- options are parsed defensively;
- missing data has an honest fallback;
- tests use the testkit;
- docs list the widget and its options;
- workspace members and registries updated.
