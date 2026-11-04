# CLI reference

`main.rs` is a hand-written dispatcher. It does not use a CLI framework.

## Dispatch behavior

- No arguments: seed assets and start the TUI.
- Recognized first argument: run the subcommand.
- Unrecognized first argument: fall through and start the TUI.
- `--help` or `-h`: print usage to stderr and exit successfully.
- `--ct` with no theme name: print usage and exit with status 2.

## Commands

| Command | Behavior |
|---|---|
| `xtop` | start the TUI system monitor |
| `xtop mcp` | seed assets, then start the MCP server on stdio |
| `xtop plugin list` | list plugins wired into the kernel manifest |
| `xtop plugin install <name>` | install a plugin from `github.com/xtop-cli/plugins` |
| `xtop plugin install <url>` | install a plugin from a git URL |
| `xtop plugin scaffold <name>` | create a plugin crate template under `plugins-dev/` |
| `xtop widget list` | list widget packs wired into the kernel |
| `xtop widget install <name>` | install a pack from `github.com/xtop-cli/widgets` |
| `xtop widget install <url-or-path>` | install a pack from a git URL or local path |
| `xtop widget scaffold <name>` | create a single-widget pack under `widgets-dev/` |
| `xtop layout check <file>` | validate a layout JSONC file |
| `xtop layout install <name>` | install a community layout from `github.com/xtop-cli/layouts` |
| `xtop --ct <theme>` | change the active theme and persist it |

## plugin install

`xtop plugin install` is a self-modifying-source workflow, not a runtime
loader.

1. Resolve the source repository: the official plugins repository for a
   name, or the given URL.
2. Clone shallowly and sparsely into a temporary directory.
3. Locate the `xtop-plugin-<name>` crate at the repo root, under
   `plugins/`, or under `crates/`.
4. Add an optional git dependency and a feature flag to the kernel
   `Cargo.toml`.
5. Run `cargo check` to verify that the manifest resolves.
6. Clean up the temporary clone.

The plugin is registered but not enabled by default. Enable it by adding the
feature to the default list or by building with `--features <feature>`.
Every installed plugin also needs a registration line in
`src/commands/share/bootstrap.rs` under its feature guard.

## widget install

`xtop widget install` edits the kernel manifest and the pack catalog:

1. Adds an optional dependency and a `widget-<name>` feature.
2. Appends a row and a registry-linking arm to
   `src/ui/layout/pack_table.rs`.
3. Runs `cargo check`.

The pack is not enabled by default. Enable the feature and select the pack
through `style.pack` or `style.widgets.<name>.pack`.

## layout install

`xtop layout install <name>`:

1. Runs a sparse, depth-1 clone of `https://github.com/xtop-cli/layouts`
   into a temporary directory.
2. Scans `layouts/custom/` for `.json` and `.jsonc` files.
3. Picks the first file whose stem or `name` matches, case-insensitively.
4. Copies the file into the user layouts directory under its original file
   name.
5. Refuses to overwrite an existing target.

`git` must be on `PATH`.

## layout check

Runs the same parser the layout crate uses at runtime
(`parse_layout_err`) and reports structural validity. It cannot know whether
widget ids will resolve in a particular kernel build.

```
xtop layout check my_layout.jsonc
```

## Theme switching

```
xtop --ct tokio
```

Persists the theme to `config.json`. Running instances poll the persisted
theme at tick boundaries and follow the change without IPC. An unknown name
fails with the list of available themes.

## Environment variables

| Variable | Effect |
|---|---|
| `XDG_CONFIG_HOME` | overrides the base Linux config directory |
| `XTOP_MAX_PROCESSES` | overrides the process snapshot cap (default 200) |
| `XTOP_WASM_DIR` | overrides the runtime WASM widget directory |
| `XTOP_EXTERNAL_DIR` | overrides the external widget descriptor directory |

## Usage output

`xtop --help` lists the commands above. The usage text is printed to stderr,
matching the existing implementation.
