# Extension contract

`xtop-extension-api` defines server-style extensions: long-running
integrations the kernel starts on demand and drives through an
`ExtensionHost` view.

The crate has zero dependencies and never depends on the kernel. It may use
the plugin contract for data-model types in the future, but no current
extension needs it, so no such dependency exists.

## Public surface

All re-exported at the crate root:

- `Extension`
- `ExtensionHost`
- `ExtensionContext`
- `ExtensionManifest`
- `ExtensionError`

## Trait

```rust
pub trait Extension: Debug + Send {
    fn manifest(&self) -> ExtensionManifest;

    fn run_server(&mut self, server_id: &str, _ctx: &mut ExtensionContext)
        -> Result<(), ExtensionError> {
        Err(ExtensionError::Unknown(format!(
            "server '{server_id}' is not provided by this extension"
        )))
    }
}
```

`manifest` is required. `run_server` defaults to an unknown-server error.
The kernel dispatches `xtop <server>` to `run_server("<server>", ...)`; the
call blocks for the lifetime of the server, which may be a read-eval-print
loop over stdio.

## Manifest

```rust
pub struct ExtensionManifest {
    pub id: String,
    pub name: String,
    pub version: String,
    pub description: String,
    pub servers: Vec<String>,
}
```

`servers` lists the server ids the extension accepts. `run_server` is the
entry point for exactly those ids.

## ExtensionHost and ExtensionContext

The kernel implements `ExtensionHost` for its live state and hands
extensions a context over it.

```rust
pub trait ExtensionHost {
    fn tick(&mut self);
    fn execute_plugin(&mut self, plugin_id: &str, action: &str, params: &str)
        -> Result<String, ExtensionError>;
}
```

`ExtensionContext` exposes `new(host)`, `tick()` and `execute_plugin(...)`.

Design intent: plugins remain the unit of domain behavior. Extensions act
through plugins instead of re-implementing domain logic. The MCP extension
follows this: every tool call maps to a `(plugin_id, action, params)` tuple
executed against the samurai plugin, optionally after a `tick()`.

The kernel maps plugin errors onto extension errors: recoverable and fatal
map one-to-one, unknown action becomes recoverable.

## ExtensionError

```rust
pub enum ExtensionError {
    Recoverable(String),
    Fatal(String),
    Unknown(String),
}
```

- `Recoverable` prints the message verbatim.
- `Fatal` prints `FATAL: <message>`.
- `Unknown` prints `unknown: <message>`.
- Implements `std::error::Error`.

The MCP server maps stdio read/write failures to `Fatal` and malformed
JSON-RPC to `Recoverable`.

## Scope: servers only

The crate documentation distinguishes two flavors:

- servers, implemented today;
- hooks that touch config, theme, layout or rendering, not implemented.

There is no hook trait, no hook registration and no hook metadata in the
manifest. Hook surface is added only when a real consumer needs it. Do not
document hooks as existing behavior.

## Authoring an extension

1. Implement `Extension`; declare server ids in `manifest()`.
2. Implement `run_server` for each declared id.
3. Read requests from the transport and drive the kernel only through
   `ExtensionContext`.
4. Forward domain work to plugins through `execute_plugin`.
5. Map every failure onto the three error variants.
6. Add tests against a mock `ExtensionHost` that records calls.
7. Add the crate to the extensions workspace.
8. Wire the kernel command behind a feature flag.
9. Document the wire protocol and the error mapping.

The reference implementation is `xtop-extension-mcp`.
