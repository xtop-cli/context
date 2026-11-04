# widgets - renderer packs

The `widgets` repository contains widget packs. A widget is a pure renderer
that draws inside a rectangle using only the read-only `WidgetState`
contract from `xtop-widget-api`.

The kernel resolves `(pack, name)` at render time, so a pack can replace a
built-in widget by name without a layout change.

## Workspace

```
widgets/
  Cargo.toml                workspace root and xtop-widgets aggregator
  xtop-widget-core/         shared engine: chart, options, formatting,
                            palette roles, frames, Painter, testkit
  xtop-widget-header/       one crate per widget
  xtop-widget-cpu/
  xtop-widget-memory/
  xtop-widget-storage/
  xtop-widget-network/
  xtop-widget-processes/
  xtop-widget-disk_io/
  xtop-widget-battery/
  xtop-widget-gpu/
  xtop-widget-summary/
  xtop-widget-sensors/
  xtop-widget-blocks/       alternative ASCII pack
  src/                      aggregator registry (xtop-widgets)
  custom/                   community packs
  docs/                     authoring guide and widget reference
```

## Registered names

Base pack:

`header`, `cpu`, `memory`, `storage`, `network`, `processes`, `disk_io`,
`battery`, `gpu`, `summary`, `sensors`.

Blocks pack:

`cpu`, `memory`, `processes`, `network`, `storage`, `disk_io`, `summary`,
`sensors`.

Names the blocks pack does not register fall back to the base pack.

## Pack selection

```json
{
  "style": {
    "pack": "blocks",
    "widgets": { "cpu": { "pack": "default" } }
  }
}
```

- global `style.pack` applies to names without an override;
- per-widget `style.widgets.<name>.pack` wins;
- a missing name in the chosen pack falls back to the base pack;
- plugin and runtime widgets keep precedence over every pack.

The blocks pack is compiled behind the kernel's `widget-blocks` feature.

## Documents

- `architecture.md` - crate model, shared engine, registry, resolution.
- `widget-reference.md` - every widget, its options and rendering rules.
- `authoring.md` - creating, registering and testing a widget crate.
- `operations.md` - local CI and installation flow.

## Local CI

```
cd ~/repos/xtop-cli/widgets
./scripts/ci.sh
```

No GitHub workflows.
