# Skill: widget authoring

## 1. Prepare

```
cd ~/repos/xtop-cli/widgets
git status
```

Decide whether the widget belongs in the base pack, the blocks pack or a
separate pack.

## 2. Create the crate

Copy an existing crate:

```
cp -r xtop-widget-processes xtop-widget-mywidget
```

or scaffold from a kernel checkout:

```
xtop widget scaffold mywidget
```

## 3. Manifest

Keep:

- `xtop-widget-api`;
- `xtop-plugin-api`;
- `xtop-widget-core`;
- `ratatui`.

Version stays `0.1.0` and inherits workspace fields.

## 4. Implement

```rust
pub fn render(f: &mut Frame, state: &dyn WidgetState, area: Rect) {
    let Some(snapshot) = state.snapshot() else { return };
    // draw only inside area
}
```

Requirements:

- early return without a sample;
- no panic on tiny areas;
- no logical-row wrapping;
- no fabricated data;
- canonical glyph helpers only;
- resolved charset and borders honored.

## 5. Options

Parse `state.widget_options()` defensively. Ignore unknown keys. Document
recognized keys in `docs/widgets.md` and in `context/widgets/widget-reference.md`.

## 6. Tests

Use `xtop-widget-core` with the `testkit` feature.

Cover:

- empty state;
- sampled state;
- small and large areas;
- every option branch;
- the standard smoke sizes: 100x34, 100x30, 80x24, 60x20, 40x15, 20x10.

## 7. Register

- add the crate to workspace `members`;
- add the dependency and registry entry to the aggregator or pack;
- for a separate pack, add it to the kernel pack table.

## 8. Verify

```
cd ~/repos/xtop-cli/widgets
./scripts/ci.sh
```

Then install into a kernel build and render the widget in a layout.

## 9. Document

- upstream `docs/widgets.md`;
- upstream `docs/authoring.md` if the process changed;
- this context: `widgets/widget-reference.md` and
  `widgets/architecture.md` if the model changed.

## Checklist

- [ ] render signature exact
- [ ] no kernel imports
- [ ] canonical glyph helpers
- [ ] options defensive
- [ ] honest empty state
- [ ] tests use testkit
- [ ] workspace and registries updated
- [ ] CI passes
- [ ] docs and context updated
