# Effect operations

## Local CI

```
cd ~/repos/xtop-cli/effects
./scripts/ci.sh
./scripts/ci.sh fmt
./scripts/ci.sh clippy
./scripts/ci.sh check
./scripts/ci.sh test
```

No GitHub workflow exists in this repository.

## Development notes

- The workspace ignores `Cargo.lock` by design.
- During local multi-repo development the effect contract may be pointed at
  `../api/crates/effect-api` temporarily; restore the floating git form
  before pushing.
- Effects are tested with synthetic buffers; no terminal is required.

## Adding a built-in effect

1. Follow `authoring.md`.
2. Add the crate to the workspace.
3. Add a kernel optional dependency and feature in the form `effect-<name>`.
4. Wire the selection through the `effect` config key.
5. Document the accepted value in `xtop/configuration.md` and the kernel
   docs.
6. Update `effects/catalog.md`, the README table and this context.

## Compatibility

- The effect contract has no registration type; the kernel holds concrete
  types.
- Contract changes must remain source compatible for existing effects or be
  recorded as breaking changes in `api/changes.md`.
- Effects must remain deterministic and must not depend on frame pacing.

## Release checklist

- CI passes across the workspace;
- the fade window and determinism tests pass;
- the kernel builds with the `effects` feature;
- the config key is documented;
- the catalog and README are current.
