# Writing an effect

## Contract recap

```rust
pub trait Effect: Send + Sync {
    fn manifest(&self) -> EffectManifest;
    fn on_frame(&mut self, buffer: &mut Buffer, elapsed: Duration);
}
```

## Host contract

- called on every rendered frame, after composition, before flush;
- `elapsed` is absolute time since the effect started;
- state may persist on `self`;
- frames are not paced; use wall-clock time, never frame counts.

## Effect guarantees

1. Stable static manifest.
2. In-place buffer rewrite.
3. No-op after the window.
4. Deterministic output for the same input.
5. `Send + Sync`.

## Pure-function pattern

Prefer deriving everything from `elapsed`:

```rust
pub const WINDOW: Duration = Duration::from_millis(500);

pub struct MyEffect;

impl Effect for MyEffect {
    fn manifest(&self) -> EffectManifest {
        EffectManifest {
            id: "my-effect",
            name: "My effect",
            description: "What it does, and over what window",
        }
    }

    fn on_frame(&mut self, buffer: &mut Buffer, elapsed: Duration) {
        if elapsed >= WINDOW {
            return;
        }
        for cell in &mut buffer.content {
            // rewrite only what must change
        }
    }
}
```

Reach for `self` state only when the visual genuinely needs history that
`elapsed` cannot express.

## Testing

```rust
let mut buffer = Buffer::empty(Rect::new(0, 0, 20, 5));
// paint cells
let before = buffer.clone();
effect.on_frame(&mut buffer, WINDOW);
assert_eq!(before.content(), buffer.content());
```

Cover the start state, a mid-window value, exactly the window, repeated
calls past the window, and an empty buffer.

## Landing an effect

1. Create `xtop-effect-<name>/` with workspace-inherited fields.
2. Depend on `ratatui` and `xtop-effect-api` through workspace dependencies.
3. Add the crate to `members` in the root manifest.
4. Implement `Effect` and document exact visual semantics.
5. Mirror the testing pattern.
6. Add a catalog entry in `docs/effects.md` and a README row.
7. Run local CI.
8. Wire the kernel feature and the `effect` config key if it is a built-in.

Do not create an empty `effects-lib/` crate.
