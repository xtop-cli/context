# WASM runtime widgets

`xtop-plugin-wasm` loads `*.wasm` modules in-process through wasmi and
registers them as plugin widgets. Guests exchange JSON defined by
`xtop-wasm-contract` with the host; they never see ratatui, the kernel or the
terminal.

## Enabling and discovery

```
cargo build --features plugin-wasm
```

At startup the host scans `<config>/wasm/` for `*.wasm` files, sorted by
path. `XTOP_WASM_DIR` overrides the directory.

Rules:

- a missing directory is normal;
- a module that fails to compile or instantiate is logged and skipped;
- a duplicate manifest name is resolved by sorted path order;
- the manifest `name`, or the file stem when empty, becomes the plugin id
  and widget name.

## ABI

A guest exports:

| Export | Signature | Meaning |
|---|---|---|
| `memory` | - | linear memory |
| `alloc` | `(i32) -> i32` | allocate bytes, return pointer or 0 |
| `dealloc` | `(i32, i32)` | free a buffer |
| `manifest` | `() -> i32` | write manifest JSON, return pointer |
| `render` | `(i32, i32) -> i32` | parse state JSON, write draw-list JSON, return pointer; 0 rejects |
| `result_len` | `() -> i32` | length of the last JSON written |

The host calls `manifest` once at load and `render` once per tick, reads
`result_len` immediately, and copies the JSON out.

The only import is:

| Import | Signature | Meaning |
|---|---|---|
| `host.log` | `(i32, i32, i32) -> ()` | log UTF-8 bytes with a level: 0 debug, 1 info, 2 warn, other error |

Any other import fails instantiation.

The `export_widget!` macro from `xtop-wasm-guest` generates the ABI glue.

## Manifest

Fields:

| Field | Default | Notes |
|---|---|---|
| `name` | file stem | widget name layouts use; must be unique |
| `version` | empty | free form |
| `description` | empty | free form |
| `author` | empty | free form |
| `max_processes` | 50 | clamped 1-4096 |
| `api` | empty | `""` or `"1"` accepted; other values warn and still load |

## Draw list

```json
{"ops":[
  {"op":"block","rect":{"x":0,"y":0,"width":30,"height":8},
   "border":"rounded","title":"CPU","fg":[200,200,200],"bg":null},
  {"op":"gauge","rect":{"x":1,"y":1,"width":28,"height":3},
   "ratio":0.42,"label":"42%","fg":[123,216,143],"bg":null,"border":null}
]}
```

Operations: `block`, `text`, `gauge`, `bar`, `sparkline`, `chart`.
Coordinates are relative to the widget area and clipped by the host.
A span is `{text, fg, bg, bold, italic, underlined, dim}` with `text`
required. Colors are `[r, g, b]` arrays and may be null or omitted.

## Sandbox

- fuel: 100,000,000 units per host-to-guest call sequence;
- memory: 64 MiB linear-memory cap per instance;
- imports: only `host.log`;
- clipping: every op rect is clipped to the widget area;
- isolation: guests cannot touch kernel types, ratatui or the terminal.

## Hot reload

Every tick the host compares the file mtime with the recorded one and
re-instantiates on change.

- a failed reload keeps the previous module and its last good draw list;
- a successful reload uses the new manifest's `max_processes` immediately;
- a changed manifest name logs a warning and layouts keep the original name;
- the `reload` debug action forces a check.

## Debug actions

| Action | Returns |
|---|---|
| `status` | name, version, path, ticks, ops, last error |
| `render` | cached draw list as JSON |
| `reload` | `{"reload":"requested"}` |

## Writing a Rust guest

```toml
[lib]
crate-type = ["cdylib"]

[dependencies]
xtop-wasm-guest = { path = "../../plugins/xtop-wasm-guest" }
```

```rust
use xtop_wasm_guest::contract::{Align, DrawList, Manifest, Op, Rect, Span, State};
use xtop_wasm_guest::export_widget;

export_widget! {
    manifest: || Manifest {
        name: "my-widget".to_string(),
        description: "hello from wasm".to_string(),
        max_processes: 1,
        ..Manifest::default()
    },
    render: |state: &State| {
        let mut list = DrawList::new();
        list.push(Op::Text {
            rect: Rect::full(state.width, state.height),
            spans: vec![Span::new(format!("tick {}", state.tick))],
            align: Align::Left,
            wrap: false,
        });
        list
    },
}
```

Build with:

```
rustup target add wasm32-unknown-unknown
cargo build --release --target wasm32-unknown-unknown
mkdir -p ~/.config/xtop/wasm
cp target/wasm32-unknown-unknown/release/my_widget.wasm ~/.config/xtop/wasm/
```

Test without the kernel:

```
cargo run -p xtop-plugin-wasm --example inspect -- path/to/my_widget.wasm
```

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| module not registered | missing export or an unsupported import |
| guest rejected the state payload | `render` returned 0 |
| draw list is not valid JSON | `result_len` does not describe the written JSON |
| fuel or trap error | runaway loop exceeded the fuel budget |
| name already loaded | duplicate manifest name; first sorted path wins |
| contract mismatch warning | manifest `api` is neither empty nor `"1"` |
| widget shows the file stem | manifest `name` is empty |
