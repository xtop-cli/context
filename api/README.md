# api - shared contracts

The `api` repository hosts the contract crates that every other repository
builds against. It never depends on the kernel or on a sibling repository.

## Workspace

```
api/
  Cargo.toml
  crates/
    plugin-api/      -> xtop-plugin-api
    widget-api/      -> xtop-widget-api
    effect-api/      -> xtop-effect-api
    extension-api/   -> xtop-extension-api
  scripts/ci.sh
  docs/
```

Workspace facts:

- resolver 2;
- version `0.1.0`, edition 2021, license MIT, `rust-version = "1.87"`;
- shared dependencies: `ratatui 0.30.2`, `serde 1` with derive,
  `serde_json 1`;
- no `tokio`, `clap` or `chrono`.

## Crates

| Crate | Package | Owns |
|---|---|---|
| `plugin-api` | `xtop-plugin-api` | data model, plugin protocol, capabilities, host state, errors, data provider, `PluginWidget` |
| `widget-api` | `xtop-widget-api` | `WidgetState`, renderer registration, glyph enums and canonical glyph helpers |
| `effect-api` | `xtop-effect-api` | `Effect` and `EffectManifest` |
| `extension-api` | `xtop-extension-api` | `Extension`, `ExtensionHost`, `ExtensionContext`, manifests and errors |

## Dependency shape

```
xtop-plugin-api   -> ratatui, serde
xtop-widget-api   -> ratatui, serde, xtop-plugin-api
xtop-effect-api   -> ratatui
xtop-extension-api -> no dependencies
```

The only internal edge is `widget-api -> plugin-api`, because `WidgetState`
uses data-model types in its signatures. The data model deliberately lives
in `plugin-api` rather than in a neutral crate.

## Consumption

All four crates are consumed as floating git dependencies:

```toml
xtop-plugin-api = { git = "https://github.com/xtop-cli/api" }
```

Consumers:

- `xtop` kernel: plugin, widget, extension and (optionally) effect APIs;
- `widgets`: widget and plugin APIs;
- `plugins`: plugin and widget APIs;
- `extensions`: extension API, plus the samurai crate for constants;
- `effects`: effect API;
- `layouts`: no api dependency by design.

The crates are not published on crates.io. Publication, tags and pinned
consumer revisions are the pending publish phase. Until then, a consumer
build resolves the remote default branch at fetch time.

## Documents

- `architecture.md` - crate graph, ownership, consumption, temporary path
  dependencies, hygiene, publish plan.
- `data-model.md` - every system model struct, provider population,
  snapshot lifecycle, `AlertThresholds`.
- `plugin-contract.md` - plugin protocol and authoring.
- `widget-contract.md` - renderer protocol, glyph helpers, pack rules.
- `extension-contract.md` - server-style extension protocol.
- `effect-contract.md` - frame effect protocol.
- `changes.md` - decision log and breaking changes.
- `operations.md` - local CI, gates, publish checklist.

## Local CI

```
cd ~/repos/xtop-cli/api
./scripts/ci.sh
./scripts/ci.sh fmt
./scripts/ci.sh clippy
./scripts/ci.sh check
./scripts/ci.sh test
```

There are no GitHub workflows in this repository.
