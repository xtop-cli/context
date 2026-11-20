# CLI reference

`src/cli/mod.rs` is a hand-written dispatcher (the binary in `src/main.rs`
only forwards to `xtop::cli::run()`). It does not use a CLI framework.

## Dispatch behavior

- No arguments: seed the base assets and start the TUI.
- Recognized first argument: run the command group or global flag.
- Unrecognized first argument: fall through and start the TUI.
- Help is layered: `xtop --help`, `xtop <group> --help`,
  `xtop <group> <command> --help`, `xtop --change-theme --help`,
  `xtop --gen-config --help`.
- Help is printed to stdout and exits 0. Usage errors are printed to stderr
  and exit 2. Runtime failures exit 1.
- The only CLI-to-TUI edge is `src/cli/commands/tui.rs`; every other command
  uses `crate::core` directly.

## Global flags

| Flag | Behavior |
|---|---|
| `--change-theme <theme>` | Persist the active theme; running TUI instances follow within one tick. `--ct` is a deprecated alias. |
| `--gen-config` | Create `config.json`, `themes/x.jsonc` and `layouts/dashboard.jsonc` when missing. |
| `-h`, `--help` | Print global help. |

## Commands

| Command | Behavior |
|---|---|
| `xtop` | Start the TUI system monitor. |
| `xtop mcp` | Seed base assets, then start the MCP server on stdio. |
| `xtop theme list` | List installed themes with source/metadata. |
| `xtop theme available` | Fetch the catalog and list themes. |
| `xtop theme current` | Print the persisted theme. |
| `xtop theme set <theme>` | Persist the active theme. |
| `xtop theme download <theme\|--all>` | Install themes from the catalog. |
| `xtop theme path` | Print the user themes directory. |
| `xtop layout list` | List effective installed layouts. |
| `xtop layout available` | Fetch the catalog and list layouts. |
| `xtop layout current` | Print the persisted layout. |
| `xtop layout set <layout>` | Persist the active layout. |
| `xtop layout download <layout\|--all>` | Install layouts from the catalog. |
| `xtop layout install <layout\|--all>` | Alias for `layout download`. |
| `xtop layout check <file>` | Validate a layout JSONC/JSON file. |
| `xtop layout path` | Print the user layouts directory. |
| `xtop config download <name\|--all>` | Download a theme or layout by name. |
| `xtop config generate` | Create the base config and assets. |
| `xtop config path` | Print config and asset directories. |
| `xtop plugin list` | List plugins wired into the kernel manifest. |
| `xtop plugin install <name\|url>` | Add a plugin git dependency and feature. |
| `xtop plugin scaffold <name>` | Create a plugin crate under `plugins-dev/`. |
| `xtop widget list` | List widget packs wired into the kernel. |
| `xtop widget install <name\|url\|path>` | Add a widget pack dependency and feature. |
| `xtop widget scaffold <name>` | Create a single-widget pack under `widgets-dev/`. |

## External assets and catalog

Only the base `x` theme and the primary `Dashboard` layout stay embedded in
the binary. Everything else is external:

- Themes live in `<config>/themes/`; layouts live in `<config>/layouts/`.
- The official `catalog.json` at the kernel repo root lists every asset with
  `id`, display `name`, `category` (layouts), `version`, `author`,
  `license`, `file`, raw `url` and `sha256`.
- `XTOP_CATALOG_URL` overrides the catalog source (HTTP(S), local path or
  `file://`).
- Downloads use `curl` (or the local file system), verify SHA-256, validate
  the JSONC (layout parser or theme parser) and write atomically.
- Installed version/author/license/checksum metadata is recorded in
  `<config>/assets.json`.
- `--force` replaces an existing file; without it, downloads skip existing
  files.

The TUI polls the persisted theme and layout name at tick boundaries. When a
downloaded theme or layout is not in the current state, it reloads the
corresponding directory, so `theme download` / `layout download` followed by
`set` works against a running instance.

## Plugin install

`xtop plugin install` is a self-modifying-source workflow, not a runtime
loader.

1. Resolve the source repository: the official plugins repository for a
   name, or the given URL.
2. Clone shallowly into a temporary directory.
3. Locate the `xtop-plugin-<name>` crate at the repo root, under
   `plugins/`, or under `crates/`.
4. Add an optional git dependency and a feature flag to the kernel
   `Cargo.toml`.
5. Run `cargo check` to verify that the manifest resolves.
6. Clean up the temporary clone.

The plugin is registered but not enabled by default. Enable it by adding the
feature to the default list or by building with `--features <feature>`.
Every installed plugin also needs a registration line in
`src/core/bootstrap.rs` under its feature guard.

## Widget install

`xtop widget install` edits the kernel manifest and the pack catalog:

1. Adds an optional dependency and a `widget-<name>` feature.
2. Appends a row and a registry-linking arm to
   `src/tui/layout/pack_table.rs`.
3. Runs `cargo check`.

The pack is not enabled by default. Enable the feature and select the pack
through `style.pack` or `style.widgets.<name>.pack`.

## Layout check

Runs the same parser the layout crate uses at runtime
(`parse_layout_err`) and reports structural validity. It cannot know whether
widget ids will resolve in a particular kernel build.

```
xtop layout check my_layout.jsonc
```

## Environment variables

| Variable | Effect |
|---|---|
| `XDG_CONFIG_HOME` | overrides the base Linux config directory |
| `XTOP_CATALOG_URL` | overrides the asset catalog source |
| `XTOP_MAX_PROCESSES` | overrides the process snapshot cap (default 200) |
| `XTOP_WASM_DIR` | overrides the runtime WASM widget directory |
| `XTOP_EXTERNAL_DIR` | overrides the external widget descriptor directory |

## Usage output

`xtop --help` lists every command on stdout. Group and subcommand help is
scoped to the selected command. Usage errors print to stderr and exit with
status 2.
