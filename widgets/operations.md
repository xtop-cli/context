# Widget operations

## Local CI

```
cd ~/repos/xtop-cli/widgets
./scripts/ci.sh
./scripts/ci.sh fmt
./scripts/ci.sh clippy
./scripts/ci.sh check
./scripts/ci.sh test
```

Stages run over the whole workspace:

- `fmt`;
- `clippy` with warnings denied;
- `check`;
- `test`.

No GitHub workflow exists in this repository.

## Adding a widget crate

1. Create `xtop-widget-<name>/`.
2. Add the manifest with the four contract dependencies.
3. Implement `render`.
4. Add tests with the `testkit` feature.
5. Add the crate to the workspace `members`.
6. Add the dependency and registry entry to `xtop-widgets`.
7. For an alternative pack, register it in that pack and document the
   fallback behavior.
8. Run CI.
9. Update `widget-reference.md` and the upstream `docs/widgets.md`.

## Adding a pack

1. Create a pack crate that exposes `registry()`.
2. Ensure it compiles against `xtop-widget-api` only.
3. Add it to the workspace and to the kernel pack catalog.
4. Add a kernel feature in the form `widget-<name>`.
5. Enable and select the pack through style configuration.
6. Document which names it overrides and which fall back.

## Testing expectations

- Every registered name renders at the standard smoke sizes.
- Charts produce multi-row glyphs when the plot is at least two rows tall.
- No row escapes the frame.
- Empty and sampled states both render.
- Option parsing is covered for each recognized key.
- Unknown options are ignored without panic.

## Blocking a release

Do not publish or promote a widget change if:

- a smoke size panics;
- a widget fabricates data for a missing source;
- the canonical glyph helpers were copied instead of imported;
- the crate imports kernel types;
- CI fails.
