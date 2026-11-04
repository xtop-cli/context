# Effect catalog

## Fade

Crate: `xtop-effect-fade`. Type: `xtop_effect_fade::FadeEffect`, a stateless
unit struct with no configuration.

### Visual behavior

The effect blends the RGB content of a rendered frame in from black.

- `elapsed == 0`: every RGB channel is zero, so the frame is black where
  widgets painted RGB colors.
- `0 < elapsed < 500 ms`: each channel is `round(channel * alpha)` with
  `alpha = elapsed / 500 ms`.
- `elapsed >= 500 ms`: full intensity.

Non-RGB colors (`Color::Reset`, indexed, grayscale) and all cell modifiers
are never touched. On a dark terminal background the whole frame fades in
from black over half a second. On a light terminal background the
`Color::Reset` backdrop is visible immediately while RGB text and backgrounds
ramp up.

### Window semantics

- window: exactly `FADE_DURATION = 500 ms`, a crate constant;
- progress is derived from the host-provided `elapsed`, so the effect never
  drifts with frame pacing;
- once `elapsed >= FADE_DURATION`, `on_frame` writes no cell;
- repeated calls past the window leave the buffer byte-identical;
- the effect is a pure function of `(buffer, elapsed)` and keeps no state.

### Tests

The crate suite covers:

- end-of-window byte identity;
- zero-elapsed dimming with non-RGB cells untouched;
- strict monotonicity at the mid-fade point;
- stable manifest metadata;
- repeated calls past the window;
- empty buffers.

## Integration

The kernel applies the active effect after layout and before flush, using
the time since the effect started. Until the kernel wiring is present in a
given build, the crate still compiles and tests standalone against the
contract.
