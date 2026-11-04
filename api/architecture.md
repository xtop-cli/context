# api architecture

The `api` repository implements the single-source-of-truth rules for the
ecosystem. Its four crates are pure contracts: types and protocols only,
with no kernel knowledge and no sibling dependency.

## Workspace layout

```
api/
  Cargo.toml
  crates/
    plugin-api/
      src/
        model.rs        shared data model
        plugin.rs       Plugin trait
        manifest.rs     PluginManifest
        context.rs      PluginContext and capability enforcement
        host.rs         HostState, RuntimeConfig, AlertThresholds
        capability.rs   PluginCapability
        error.rs        PluginError
        provider.rs     SystemDataProvider
        widget.rs       PluginWidget
        color.rs        hex_to_rgb
        lib.rs
    widget-api/
      src/
        state.rs        WidgetState trait
        renderer.rs     WidgetRenderer, WidgetRegistration
        glyph.rs        ChartCharset, WidgetBorders, helpers
        lib.rs
    effect-api/
      src/lib.rs        Effect, EffectManifest
    extension-api/
      src/
        context.rs
        error.rs
        extension.rs
        host.rs
        manifest.rs
        lib.rs
  docs/
  scripts/ci.sh
```

## Ownership rules

| Rule | Surface | Location |
|---|---|---|
| DR-1 | data model and plugin protocol | `crates/plugin-api/src/` |
| DR-2 | widget render contract and glyph mapping | `crates/widget-api/src/` |
| DR-4 | extension protocol | `crates/extension-api/src/` |
| DR-5 | effect protocol | `crates/effect-api/src/` |

DR-3 states that layouts have no api surface: layout files address widgets
by plain string names, and `xtop-layout` lives in the `layouts` repository
without any api dependency. DR-6 places ecosystem constants at the producer:
the samurai plugin id and action names are exported by
`xtop-plugin-samurai`, not by `api`. Both rules keep the contract surface
small.

## Boundary types

- `xtop_widget_api::WidgetRegistration` renders over `&dyn WidgetState`.
- `xtop_plugin_api::PluginWidget` renders over `&dyn HostState`.
- The two are deliberately distinct and must not share a name. A grep gate
  checks that `WidgetRegistration` does not appear outside `widget-api`.
- `AlertThresholds` is defined once, in `plugin-api`, and re-exported at
  the crate root. A grep gate checks that no other definition exists.

## Consumption model

Every consumer declares floating git dependencies without `rev` or `tag`:

```toml
xtop-widget-api = { git = "https://github.com/xtop-cli/api" }
```

Consequences:

- a consumer sees whatever the remote default branch contains when Cargo
  fetches;
- local working-tree changes in `api` are invisible to siblings until
  pushed;
- reproducibility is bounded by the last fetched revision;
- nothing can break because of a tag or publish change, because no tags or
  publications exist.

## Temporary path-dependency pattern

Two forms are used for local multi-repo development.

Per-repository override with a patch:

```toml
[patch."https://github.com/xtop-cli/api"]
xtop-widget-api = { path = "../api/crates/widget-api" }
xtop-plugin-api = { path = "../api/crates/plugin-api" }
```

Direct path dependency when there is no git declaration to patch:

```toml
xtop-plugin-api = { path = "../api/crates/plugin-api" }
```

Absolute `file://` URLs are equivalent. Relative `../api/...` paths are the
norm because the repositories are siblings.

Combined verification wires all repositories into a scratch workspace with
path dependencies, then runs tests and clippy across the whole ecosystem.
The scratch workspace lives outside the repositories or in a git-ignored
directory.

Mandatory rule: every manifest is restored to its floating git dependency
form before the owner pushes. A `[patch]` section is preferred because
restoring means deleting the section.

## Repository hygiene

- Local CI only: `./scripts/ci.sh [fmt|clippy|check|test]`.
- No GitHub workflows.
- `cargo doc --workspace --no-deps` must build without warnings.
- Grep gates:
  - no `WidgetRegistration` outside `widget-api`;
  - no `AlertThresholds` definition outside `plugin-api`;
  - no `PluginWidgetFn` alias;
  - no `plugins_dir_tmp` references;
  - no Docker symbols.
- Tests cover capability gating, `top_processes` ordering, the
  `AlertThresholds` serde contract, the glyph mapping tables and a
  fake-frame effect test.

## Publish plan

Publication is pending. The intended order is:

1. `api` first, because every other repository depends on it.
2. Publish the four crates to crates.io under `0.1.0` or a chosen release
   version.
3. Tag the repository.
4. Pin or switch consumers to published versions.
5. Add per-repository release automation.

Until step 4, consumers keep floating git dependencies. A partial migration
(pinning one consumer while others float) is discouraged.

## Adding a contract type

1. Decide which crate owns it using the ownership rules above.
2. Prefer additive changes with default implementations.
3. Document the semantics and the host obligations in the crate docs and in
   the matching contract document under `docs/`.
4. Add a test that pins serde shape or trait behavior when the type crosses
   a persistence or wire boundary.
5. Record breaking changes in `docs/changes.md`.
6. Run local CI.
7. Update consumers and this context repository.
