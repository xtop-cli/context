# Effect contract

`xtop-effect-api` defines frame effects: stateful, full-buffer visual
transitions applied to every rendered frame, after widgets and layout are
drawn and before the terminal flush.

The crate is intentionally minimal: two public items. It depends only on
`ratatui` for the buffer type and never on the kernel.

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

- `EffectManifest` is plain static data and derives `Debug, Clone,
  PartialEq, Eq`.
- `Effect` requires `Send + Sync`.
- An effect owns its progress state on `self`.

There is no registration glue type. It is added only when the kernel wiring
needs it.

## Host invocation contract

1. `on_frame` runs on every rendered frame, after composition and before
   flush. The effect may rewrite any cell.
2. `elapsed` is the time since the effect started, not the frame delta.
3. Effects are stateful across frames.
4. The host does not pace frames. The kernel ticks at roughly one second
   plus ad-hoc repaints. Transitions must advance by wall-clock `elapsed`,
   never by frame count alone.

The effect sees the final composed picture, overlays included, and rewrites
it in place.

## Implementing an effect

An effect is a struct implementing the trait. It should:

1. return stable static metadata from `manifest()`;
2. rewrite cells in place;
3. become a no-op after its window, leaving later frames byte-identical;
4. be deterministic for a given `(buffer, elapsed)` pair.

When semantics allow, keep the effect a pure function of `(buffer, elapsed)`
instead of accumulating progress on `self`. That makes it testable without a
time source.

## Testing

Build a buffer with `Buffer::empty(Rect::new(0, 0, w, h))`, paint cells
through the public `content` field, call `on_frame`, and compare
`buffer.content()`.

Cover at least:

- `elapsed == 0`;
- a mid-window value;
- exactly the window;
- repeated calls past the window (stability);
- an empty buffer.

`Buffer: Clone` makes byte-identity checks one line.

## Status

- Contract: implemented.
- First implementation: `xtop-effect-fade` in the `effects` repository.
- Kernel wiring: optional `effects` feature and the `effect` config key.
- No publication to crates.io.

No effect may assume a pacing guarantee beyond this contract.
