# Plugin operations

## Local CI

```
cd ~/repos/xtop-cli/plugins
./scripts/ci.sh
./scripts/ci.sh fmt
./scripts/ci.sh clippy
./scripts/ci.sh check
./scripts/ci.sh test
```

Optional stages:

```
./scripts/ci.sh wasm
./scripts/ci.sh external
```

- `wasm` builds the Rust guests and the C examples, then smoke-tests each
  module with the host loader. It requires `wasm32-unknown-unknown`, clang
  and wasm-ld.
- `external` drives the Lua, Python and Node examples end to end and skips
  runtimes that are not installed.

No GitHub workflow exists in this repository.

## Building examples

```
./scripts/build-examples.sh
./scripts/build-examples.sh --check
```

Artifacts land under
`examples/wasm/target/wasm32-unknown-unknown/release/*.wasm` and
`examples/wasm/c-*/*.wasm`.

## Adding a native plugin

1. Create `plugins/xtop-plugin-<name>/`.
2. Depend on `xtop-plugin-api`.
3. Implement `Plugin`.
4. Keep the version sourced from the crate.
5. Add tests with a `HostState` double.
6. Add the crate to the workspace members.
7. Run CI.
8. Install into the kernel with `xtop plugin install <name>`.
9. Add the registration line under the feature guard in the kernel.
10. Document actions and capabilities.

## Adding a runtime widget example

WASM:

1. Add a `cdylib` crate under `examples/wasm/`.
2. Depend on `xtop-wasm-guest`.
3. Implement `manifest` and `render`.
4. Add it to `examples/wasm/Cargo.toml`.
5. Extend the build and test scripts if needed.

External:

1. Add a directory under `examples/external/`.
2. Provide a descriptor and the helper program.
3. Keep stdout JSON-only.
4. Document the descriptor.

## Security checklist

- Capability checks remain intact.
- External widgets are described as unsandboxed.
- WASM limits remain enforced: fuel, memory, allowed imports, clipping.
- No secret is logged.
- Process termination stays restricted to the current user.

## Compatibility notes

- The shared contract ABI is `"1"`.
- Contract changes must remain backward compatible.
- A manifest `api` value other than empty or `"1"` logs a warning and still
  loads.
- The external and WASM hosts share the state and draw-list definitions; a
  change to one affects both.
