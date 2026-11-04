# layouts - data-driven arrangement

The `layouts` repository provides `xtop-layout`, the crate that owns the
layout model, the JSONC loader, the layout modes and the shipped layouts.
The kernel consumes it as a git dependency and translates the tree into
ratatui constraints at render time.

The crate deliberately has no UI dependency and no `xtop-*` contract
dependency. It is pure data, serde parsing and small mode rules.

## Workspace

```
layouts/
  Cargo.toml
  src/
    lib.rs        public exports
    model.rs      LayoutDef, LayoutNode, LayoutArea, constraints
    loader.rs     JSONC stripping, parsing, merging, embedded defaults
    mode.rs       LayoutMode and size-based degradation
  layouts/
    default/      10 embedded layouts
    custom/       community layouts
  docs/
```

## Shipped defaults

Seven mode-bound layouts:

- `dashboard`
- `vertical`
- `horizontal`
- `cpu_focus`
- `memory_focus`
- `network_focus`
- `process_focus`

Three detail presets:

- `detail_dashboard`
- `detail_network`
- `detail_processes`

The first seven map to modes in fixed order. The three presets are appended
after them and are addressed by name.

## Personalization model

1. Defaults are compiled into the binary and copied as templates into the
   user layouts directory on first run.
2. A user file whose `name` matches a default replaces it in place and keeps
   its palette slot.
3. A file with a new name is appended as an additional layout.
4. Community layouts live under `layouts/custom/` and are installed with
   `xtop layout install <name>` or copied manually.

## Documents

- `schema.md` - the complete JSONC schema and tiling rules.
- `authoring.md` - writing, validating, activating and sharing layouts.
- `defaults.md` - structure of the ten embedded layouts.
- `operations.md` - local CI, adding a default, review checklist.

## Local CI

```
cd ~/repos/xtop-cli/layouts
./scripts/ci.sh
```

No GitHub workflows.
