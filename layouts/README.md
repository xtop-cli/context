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
    loader.rs     JSONC stripping, parsing, merging, base layout
    mode.rs       LayoutMode and size-based degradation
  layouts/
    default/      10 JSONC layout files (only Dashboard is embedded)
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

Only `Dashboard` is embedded in the crate. The other mode layouts and the
three presets are catalog assets addressed by name.

## Personalization model

1. The embedded Dashboard is copied as a template into the user layouts
   directory on first run.
2. Every other shipped layout is installed from `catalog.json` with
   `xtop layout download <name>` (or `xtop config download --all`).
3. A user file whose `name` matches an installed layout replaces it in
   place.
4. A file with a new name is appended as an additional layout.
5. Community layouts live under `layouts/custom/`; they are catalog assets
   installed with `xtop layout download <name>` or copied manually.

## Documents

- `schema.md` - the complete JSONC schema and tiling rules.
- `authoring.md` - writing, validating, activating and sharing layouts.
- `defaults.md` - structure of the shipped layout files and the base/ catalog split.
- `operations.md` - local CI, adding a default, review checklist.

## Local CI

```
cd ~/repos/xtop-cli/layouts
./scripts/ci.sh
```

No GitHub workflows.
