# plugins - functionality and runtime widgets

The `plugins` repository hosts plugin implementations and the two runtime
widget hosts. Every crate builds against `xtop-plugin-api` and the shared
widget contract; the kernel consumes them as git dependencies.

## Workspace

```
plugins/
  plugins/
    xtop-plugin-samurai/    native plugin: monitoring, process queries,
                            heuristic alerts, one widget
    xtop-plugin-wasm/       runtime host: sandboxed WASM modules
    xtop-plugin-external/   runtime host: helper processes
    xtop-wasm-contract/     serializable manifest, state and draw list
    xtop-widget-replay/     host-side draw-list replay
    xtop-wasm-guest/        Rust guest SDK with export_widget!
  examples/
    wasm/                   Rust and C guests
    external/               Lua, Python and Node widgets
  docs/
  scripts/
```

## Plugins

- `xtop-plugin-samurai` is the reference plugin. It exposes 12 JSON actions,
  runs 10 heuristic rules over the process table and registers one widget
  named `samurai`.
- A plugin is compile-time: the kernel enables it with a Cargo feature and a
  git dependency. A plain kernel build does not need this repository.

## Runtime widgets

Two optional hosts open the widget surface to code that is not compiled into
the kernel:

- `xtop-plugin-wasm` loads sandboxed `.wasm` modules in-process with wasmi.
- `xtop-plugin-external` spawns helper processes and speaks line-delimited
  JSON.

Both register through the plugin widget path, so they take precedence over
every compiled-in pack and can replace any widget name. Neither feature is
part of the default kernel build.

## Documents

- `architecture.md` - samurai lifecycle, action API, analysis and widget.
- `samurai.md` - detailed action and params reference.
- `rules.md` - the ten heuristic detection rules.
- `runtime-widgets.md` - shared runtime widget model and security notes.
- `wasm-widgets.md` - WASM ABI, sandbox, hot reload and authoring.
- `external-widgets.md` - descriptor format, protocol and lifecycle.
- `operations.md` - local CI, examples and extension checklist.

## Local CI

```
cd ~/repos/xtop-cli/plugins
./scripts/ci.sh                 # fmt, clippy, check, test
./scripts/ci.sh wasm            # build guests and smoke-test the host
./scripts/ci.sh external        # drive Lua/Python/Node examples
```

The optional stages need extra toolchains; the default stages do not.
