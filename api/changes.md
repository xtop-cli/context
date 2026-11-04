# Decision log and breaking changes

Summary of the decisions that shaped the contract repository and the changes
consumers had to absorb. The full text lives in `api/docs/changes.md`.

## Ecosystem decisions DR-1 to DR-7

| Rule | Meaning in `api` |
|---|---|
| DR-1 | data model and plugin protocol exist once in `plugin-api`; no other `AlertThresholds` definition is allowed |
| DR-2 | widget render contract exists once in `widget-api`; plugin widgets use the distinct `PluginWidget` type |
| DR-3 | no layout surface in `api`; layouts use string widget names |
| DR-4 | extension types exist once in `extension-api` |
| DR-5 | effect contract exists once in `effect-api` |
| DR-6 | ecosystem constants live at the producer, not in `api` |
| DR-7 | dependency and toolchain policy: ratatui 0.30.2, serde 1, rust 1.87, no tokio/clap/chrono, floating consumer git dependencies |

## Milestone decisions D1 to D8

- D1: capability gating forces `Result` reads for `snapshot`, `system_info`
  and `top_processes`.
- D2: `top_processes` sorts by CPU descending with `total_cmp` and then
  truncates; the sort is stable.
- D3: border mapping follows ratatui 0.30.2 semantics. `Native` maps to
  `border::PLAIN`; `Plain` and `Ascii` map to the canonical `ASCII_BORDER`;
  ratatui no longer ships an ASCII set.
- D4: no `EffectRegistration` glue type until a consumer exists.
- D5: glyph mapping helpers live in `glyph.rs` and are not re-exported at
  the crate root; only the enums are.
- D6: every member manifest opts into
  `rust-version.workspace = true` so the MSRV is enforced per package.
- D7: the plugin widget type was renamed conceptually without printing the
  old symbol, keeping the grep gate green.
- D8: `AlertThresholds` does not derive `Default`; the kernel owns config
  defaults.

## Breaking changes

| Change | Consumer impact | Migration |
|---|---|---|
| `WidgetRegistration` renamed to `PluginWidget` in the plugin contract | kernel imports the old plugin type; samurai names it | kernel imports `PluginWidget`; delete the alias; update samurai |
| plugin reads return `Result` | samurai calls `ctx.snapshot()` and reads directly | handle `Err` at every read site |
| Docker surface removed | kernel still assigned an empty `dockers` field and overrode `docker_info` | remove both kernel references |
| glyph helpers canonicalized | base and blocks packs re-implemented color, border and marker mapping | packs import `xtop_widget_api::glyph` helpers; delete private copies |
| ratatui 0.30.2 in api while consumers pinned 0.29 | duplicated ratatui versions until aligned | align kernel, widgets and samurai |
| rust-version 1.87 and edition 2021 | older toolchains fail | bump toolchains |
| `AlertThresholds` becomes the serde contract type with plain keys | kernel had a local copy | delete the copy, persist the contract type, keep JSON keys |
| `data_dir()` documentation no longer prescribes a path | doc-only | none |
| glyph enums keep their serde names | none expected | none |

## Rule for future changes

Record every breaking change here and in `api/docs/changes.md`, update every
consumer in the same effort, and state the migration path. Silent breakage
is not acceptable for contracts.
