# effects - frame effects

The `effects` repository hosts frame effects for the kernel. An effect
receives the fully rendered ratatui buffer plus the time since it started,
and rewrites the buffer in place before the terminal flush.

Effects implement `xtop-effect-api` from the `api` repository.

## Workspace

```
effects/
  Cargo.toml
  xtop-effect-fade/    fade-in from black over 500 ms
  docs/
    effects.md         catalog
    write-effect.md    authoring guide
```

Each effect lives in its own `xtop-effect-<name>/` crate. Shared helpers
would live in an `effects-lib/` crate, created only when two effects actually
share code.

## Current effect

| Crate | Manifest id | Behavior |
|---|---|---|
| `xtop-effect-fade` | `fade` | fades a rendered frame in from black over 500 ms, deterministic, zero configuration |

## Kernel integration

The kernel wires built-in effects behind the optional `effects` feature and
the `effect` config key. The draw closure applies the active effect after
layout and before flush, passing the time since the effect started.

## Documents

- `architecture.md` - contract and workspace layout.
- `catalog.md` - exact fade semantics.
- `authoring.md` - writing, testing and landing an effect.
- `operations.md` - local CI and release notes.

## Local CI

```
cd ~/repos/xtop-cli/effects
./scripts/ci.sh
```

No GitHub workflows.
