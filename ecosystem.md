# Ecosystem architecture

The `xtop-cli` organization splits one application into a thin kernel and a
set of independently compilable repositories. The split is about ownership
and testability, not about runtime services: everything ships as Rust crates
linked into a single binary, plus one optional documentation website.

## Repository roles

```
+---------------------------------------------------------------+
|                            api                                |
|  xtop-plugin-api  xtop-widget-api                             |
|  xtop-extension-api  xtop-effect-api                          |
+----------------+----------------+----------------+------------+
                 |                |                |
     +-----------+--+   +---------+----+   +-------+---------+
     |    xtop      |   |   widgets    |   |    layouts      |
     |   (kernel)   |   |  (renderers) |   |  (arrangement)  |
     +------+-------+   +--------------+   +-----------------+
            |
     +------+-------+   +--------------+   +-----------------+
     |   plugins    |   |  extensions  |   |     effects     |
     | (samurai,    |   |  (MCP server)|   |  (fade)         |
     |  wasm,       |   +--------------+   +-----------------+
     |  external)   |
     +--------------+
            |
     +------+-------+
     |     web      |
     | (site, docs) |
     +--------------+
```

Legend:

- `api` defines the contracts. It depends only on `ratatui` and `serde`; it
  never depends on the kernel.
- The kernel depends on the contract crates and consumes the ecosystem
  crates as optional features.
- `widgets`, `layouts`, `plugins`, `extensions` and `effects` depend on
  `api` only (MCP additionally depends on the samurai plugin crate for its
  constants).
- `web` has no compile-time dependency on the Rust crates. It mirrors their
  Markdown documentation at build time.

## Why the split exists

The kernel started as a multi-crate workspace, then became a monocrate with
plugins inside. Each customization axis was extracted so that:

- a contributor can ship a widget pack, layout, plugin, extension or effect
  without editing kernel code;
- every area compiles and tests in isolation against contracts;
- the kernel stays a thin host for metrics, state, layout rendering and
  command dispatch.

The split is recorded as phases F0 through F5 in `xtop/docs/multi-repo.md`:

| Phase | State | Content |
|---|---|---|
| F0 | done | organization, repositories, sibling checkouts |
| F1 | done | contract crates extracted into `api` |
| F2 | done | samurai moved to `plugins`; MCP moved to `extensions` |
| F3 | done | effect workspace and fade effect |
| F4 | done | server-style extension contract and MCP |
| F5 | pending | crates.io publication, tags, pinned dependencies, per-repo CI and kernel releases |

## Contract graph

```
xtop-plugin-api   -> ratatui, serde
xtop-widget-api   -> ratatui, serde, xtop-plugin-api (path inside api)
xtop-effect-api   -> ratatui
xtop-extension-api -> no dependencies
```

Only one edge exists between the four contract crates: the widget contract
names data-model types from the plugin contract. The effect and extension
contracts are standalone.

Consumer edges today:

| Consumer | Contract edges | Ecosystem edges |
|---|---|---|
| `xtop` | plugin-api, widget-api, extension-api, effect-api (optional) | widgets, layouts, plugins (optional), extensions (optional), effects (optional) |
| `widgets` | widget-api, plugin-api | none |
| `layouts` | none | none |
| `plugins` | plugin-api, widget-api | none |
| `extensions` | extension-api | plugin-samurai (compile-time for constants) |
| `effects` | effect-api | none |

All consumer edges are floating git dependencies during the current cycle.
The final intended state is published crates plus pinned revisions; that is
deferred to F5.

## Integration modes

| Mode | Mechanism | Current use |
|---|---|---|
| Compile-time | Cargo features plus git dependencies | samurai, MCP extension, blocks pack, fade effect |
| Development-time | `xtop plugin install <name or url>` clones a repo, edits the kernel manifest, runs `cargo check` | first steps with a third-party plugin repo |
| Runtime widgets | `plugin-wasm` and `plugin-external` features load code from the user config directory | widgets in Rust/WASM, C, Lua, Python, Node without recompiling the kernel |
| Future runtime ABI | native dynamic discovery of `xtop-plugin-*` and friends | explicitly deferred, not implemented |

The default kernel build enables `plugin-samurai` and `mcp-extension`. The
`widget-blocks`, `effects`, `plugin-wasm` and `plugin-external` features are
opt-in. `cargo build --no-default-features` yields the pure core.

## Data flow of one monitoring tick

1. The kernel ticks at `update_interval_ms` (default 1000 ms).
2. The sysinfo provider refreshes metrics and records byte-counter baselines
   for rate computation.
3. The provider assembles one `SystemSnapshot`.
4. `AppState` caches the snapshot; widgets and actions in that frame read
   the same cache. The render path never resamples.
5. History buffers (per-core CPU, memory, network rates, disk rates, load)
   are pushed from that sample.
6. Plugin tick handlers run against `PluginContext`; plugin data providers
   are merged through `CompositeProvider`.
7. Extensions act only through `ExtensionHost`: `tick()` advances the
   monitoring tick and `execute_plugin()` invokes a hosted plugin action.
   Extensions do not implement domain behavior themselves.

## Render resolution order

When the layout engine draws a widget name:

1. Plugin widgets are checked first, so runtime widgets and plugin widgets
   can replace any name.
2. The pack selected by the user's style configuration
   (`style.widgets.<name>.pack`, then `style.pack`) is consulted.
3. The base pack `xtop-widgets` is used as the fallback.

If no renderer matches, the area is left empty and the kernel emits one
stderr warning per unknown name per process.

## Extensibility axes

| Axis | Unit | Contract | Repository |
|---|---|---|---|
| Visual renderer | widget crate | widget-api | `widgets` |
| Arrangement | JSONC layout | layout format (string names) | `layouts` |
| Domain functionality | plugin crate | plugin-api | `plugins` |
| Server integration | extension crate | extension-api | `extensions` |
| Frame post-processing | effect crate | effect-api | `effects` |
| Presentation site | Next.js app | none | `web` |

## Distribution

- `cargo install --git https://github.com/xtop-cli/xtop --all-features`
- `install.sh` for Linux distributions (builds from source)
- `install.ps1` for Windows (requires Rust)
- AUR package `xtop-git` tracks the `xtop` repository HEAD
- The website is a static Next.js export deployed to GitHub Pages

There are no tagged releases in the upstream repositories at the current
snapshot, and no crates.io publication. Version strings remain `0.1.0`.
