# Effect architecture

An effect is a visual post-process over the final composed frame. The
contract lives in `xtop-effect-api`; implementations live in this
repository.

## Contract

```rust
pub struct EffectManifest {
    pub id: &'static str,
    pub name: &'static str,
    pub description: &'static str,
}

pub trait Effect: Send + Sync {
    fn manifest(&self) -> EffectManifest;
    fn on_frame(&mut self, buffer: &mut ratatui::buffer::Buffer, elapsed: std::time::Duration);
}
```

Two public items, no registration glue type. The kernel holds concrete
effects behind the optional `effects` feature.

## Host obligations

- `on_frame` runs on every rendered frame, after composition and before
  terminal flush.
- `elapsed` is the absolute time since the effect started, not a frame
  delta.
- Effects may keep state on `self` across frames.
- The host does not pace frames. The kernel ticks around once per second
  plus ad-hoc repaints. Effects must advance by wall-clock `elapsed`, never
  by counting frames.

## Effect obligations

1. Return stable static metadata.
2. Rewrite any cells in place.
3. Become a no-op after the window, leaving later frames byte-identical.
4. Be deterministic for a given `(buffer, elapsed)` pair.
5. Stay `Send + Sync`.

## Workspace layout

```
effects/
  Cargo.toml               workspace root
  xtop-effect-fade/        the first and only effect crate
  docs/
    effects.md
    write-effect.md
```

Each effect is an independent crate. A shared helper crate is added only
when a real second consumer exists.

## Kernel wiring

The kernel's optional `effects` feature pulls `xtop-effect-fade` and
`xtop-effect-api`. The `effect` config key selects the active effect:
`"fade"` activates the fade; any other value disables effects. Builds
without the feature carry no extra dependencies.

## Distribution

The crate is consumed as a floating git dependency from the kernel. It is not
published to crates.io.
