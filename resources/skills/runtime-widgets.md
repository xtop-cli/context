# Skill: runtime widgets

Use this to build and install a widget that is not compiled into the kernel.

## Choose the host

- WASM: sandboxed, in-process, Rust or C, hot reload, no OS access.
- External: any language, process isolation, runs with user privileges, no
  hot reload.

## Build a WASM guest

1. Create a `cdylib` crate depending on `xtop-wasm-guest`.
2. Write `src/lib.rs` with `export_widget!`.
3. Build:

```
rustup target add wasm32-unknown-unknown
cargo build --release --target wasm32-unknown-unknown
```

4. Test without the kernel:

```
cd ~/repos/xtop-cli/plugins
cargo run -p xtop-plugin-wasm --example inspect -- path/to/widget.wasm
```

5. Install:

```
mkdir -p ~/.config/xtop/wasm
cp path/to/widget.wasm ~/.config/xtop/wasm/
```

6. Build the kernel with `--features plugin-wasm`.

## Build an external widget

1. Create a helper program in any language.
2. Write a descriptor under `external/`:

```json
{
  "name": "my-widget",
  "command": ["python3", "/absolute/path/widget.py"],
  "timeout_ms": 2000,
  "max_processes": 1
}
```

3. The program must:
   - answer `{"type":"manifest"}` with a manifest;
   - answer `{"type":"render","state":...}` with a draw list;
   - write only JSON to stdout;
   - send diagnostics with `log` messages or stderr;
   - exit cleanly on shutdown or EOF.
4. Build the kernel with `--features plugin-external`.

## Reference in a layout

```jsonc
{
  "name": "Runtime Demo",
  "root": {
    "direction": "vertical",
    "areas": [
      { "widget": "header", "size": 3 },
      { "widget": "my-widget", "size": "*" }
    ]
  }
}
```

## Limits and behavior

- WASM: 100M fuel, 64 MiB memory, only `host.log`, rect clipping, hot
  reload.
- External: per-response timeout, no hot reload, no sandbox.
- Both: `max_processes` caps the process list after a CPU-descending sort.
- Both: the guest is called per tick; frames replay the cached draw list.

## Debug

Plugin actions `status`, `render` and `reload` are reachable through
`execute_plugin`. `reload` is unavailable for external widgets.

## Checklist

- [ ] correct host feature compiled in
- [ ] discovery directory or override set
- [ ] unique widget name
- [ ] state and draw list valid JSON
- [ ] no non-JSON stdout
- [ ] timeouts acceptable
- [ ] layout references the name
- [ ] docs updated
