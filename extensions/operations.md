# Extension operations

## Local CI

```
cd ~/repos/xtop-cli/extensions
./scripts/ci.sh
./scripts/ci.sh fmt
./scripts/ci.sh clippy
./scripts/ci.sh check
./scripts/ci.sh test
```

No GitHub workflow exists in this repository.

## Build

```
cd ~/repos/xtop-cli/extensions
cargo build --workspace
```

## Adding an extension

1. Create `extensions/xtop-extension-<name>/`.
2. Depend on `xtop-extension-api`; add the plugin contract or a plugin crate
   only when the extension needs them.
3. Implement `Extension`: `manifest()` and `run_server()`.
4. Drive the kernel only through `ExtensionContext`.
5. Delegate domain behavior to plugins through `execute_plugin`.
6. Map failures onto `Recoverable`, `Fatal` or `Unknown`.
7. Add tests with a mock `ExtensionHost`.
8. Add the crate to the workspace members.
9. Wire the kernel subcommand behind a feature flag.
10. Document the wire protocol and error mapping.

## Testing the MCP server

Unit tests run without a terminal:

```
cargo test -p xtop-extension-mcp
```

Manual smoke test:

```
printf '%s\n' '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{}}' | cargo run -p xtop-extension-mcp
```

End-to-end through the kernel:

```
cargo run -- mcp
```

The kernel build must include `mcp-extension` and `plugin-samurai`.

## Compatibility checklist

- tool names stay derived from action constants;
- no hardcoded plugin id remains in the extension;
- error mapping uses the documented JSON-RPC codes;
- malformed JSON behavior stays documented;
- tool schemas match the params translation;
- the extension builds against the current plugin action set.

## Security notes

- The MCP server is a local stdio process. It does not open a network port.
- Tool calls can kill processes and modify configuration, subject to the
  plugin capabilities.
- Do not expose the server to untrusted clients without an authorization
  layer; none exists in the protocol implementation.
