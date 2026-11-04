# extensions - server-style integrations

The `extensions` repository hosts optional behaviors that the kernel starts
on demand and drives through the extension host contract. Today that means
server-style extensions.

## Workspace

```
extensions/
  Cargo.toml
  extensions/
    xtop-extension-mcp/    MCP server over JSON-RPC 2.0 stdio
  docs/
    architecture.md
    mcp-protocol.md
```

## Model

- An extension implements `Extension` from `xtop-extension-api`.
- `manifest()` declares identity and the server ids it provides.
- `run_server(server_id, ctx)` runs the server loop.
- The extension drives the kernel only through `ExtensionContext::tick` and
  `ExtensionContext::execute_plugin`.
- Domain behavior stays in plugins; extensions delegate through
  `execute_plugin`.

The kernel works fully without any extension.

## Current extension

`xtop-extension-mcp`:

- launched by the kernel as `xtop mcp`;
- exposes the samurai plugin as 12 MCP tools;
- builds its tool table from the plugin's exported constants, so a rename
  breaks the build instead of breaking AI clients;
- maps plugins errors to JSON-RPC errors.

## Documents

- `architecture.md` - MCP extension structure and the samurai coupling.
- `mcp-protocol.md` - transport, methods, tool table, error mapping.
- `operations.md` - build, CI, adding an extension.

## Local CI

```
cd ~/repos/xtop-cli/extensions
./scripts/ci.sh
```

No GitHub workflows.
