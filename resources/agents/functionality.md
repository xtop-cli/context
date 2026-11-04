# Agent: functionality

## Scope

The `plugins`, `extensions` and `effects` repositories: samurai, runtime
widget hosts, MCP server and frame effects.

## Required reading

- `../../plugins/README.md`
- `../../plugins/architecture.md`
- `../../plugins/samurai.md`
- `../../plugins/rules.md`
- `../../plugins/runtime-widgets.md`
- `../../extensions/architecture.md`
- `../../extensions/mcp-protocol.md`
- `../../effects/architecture.md`
- `../../effects/catalog.md`
- `../../api/plugin-contract.md`, `../../api/extension-contract.md`,
  `../../api/effect-contract.md`
- `../../conventions.md`

## Workflow

Plugins:

1. Scaffold or create the crate in the plugins workspace.
2. Implement `Plugin`; declare only used capabilities.
3. Keep the version sourced from the crate.
4. Add tests with a `HostState` double.
5. Add the crate to the workspace and run CI.
6. Wire into the kernel behind a feature and register it in
   `src/commands/share/bootstrap.rs`.
7. Document actions and params.

Extensions:

1. Implement `Extension`.
2. Drive the kernel only through `ExtensionContext`.
3. Delegate domain behavior to plugins.
4. Test with a mock `ExtensionHost`.
5. Wire the kernel subcommand.

Effects:

1. Implement `Effect`.
2. Prefer a pure function of `(buffer, elapsed)`.
3. Become a no-op after the window.
4. Test with synthetic buffers.

## Guardrails

- No duplicated action constants; import them from the plugin.
- No extension that re-implements domain logic.
- No external widget described as sandboxed.
- No WASM host without fuel, memory and import limits.
- No effect that depends on frame pacing.
- No secret logging.

## Verification

```
cd ~/repos/xtop-cli/plugins    && ./scripts/ci.sh
cd ~/repos/xtop-cli/extensions && ./scripts/ci.sh
cd ~/repos/xtop-cli/effects    && ./scripts/ci.sh
```

Optional runtime host checks:

```
cd ~/repos/xtop-cli/plugins && ./scripts/ci.sh wasm
cd ~/repos/xtop-cli/plugins && ./scripts/ci.sh external
```

## Output

Report the actions, protocols or effects added, capability changes, the
kernel wiring, the tests run and the security properties preserved.
