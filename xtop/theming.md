# Theming

Themes are JSONC files with an explicit background and foreground plus a
16-entry palette. Every slot has a fixed role; renderers select colors by
role, never by taste.

## Format v2

```jsonc
{
  "name": "my-theme",
  "background": "#1a1b1c",
  "foreground": "#abb2bf",
  "palette": [
    "#1a1b1c", "#e06c75", "#98c379", "#e5c07b",
    "#d19a66", "#c678dd", "#56b6c2", "#abb2bf",
    "#3e4451", "#e06c75", "#98c379", "#e5c07b",
    "#d19a66", "#c678dd", "#56b6c2", "#abb2bf"
  ]
}
```

Rules:

- `name` is the identity used by configuration and the CLI.
- Colors are hex strings with an optional `#`.
- `//` and `/* */` comments are supported.
- The explicit background and foreground are optional only for legacy files
  written against the old 16-slot format; absent values fall back to slot 0
  and slot 7.
- Slots must not be repurposed.

## Roles

| Slot | Role | Typical use |
|---|---|---|
| 0 | background alias | legacy frames |
| 1 | alert | high fills, average CPU line |
| 2 | good | normal fills, RAM line, battery |
| 3 | warn | mid gradient stop |
| 4 | read / RX | network RX, disk reads |
| 5 | write / TX | network TX, disk writes, GPU |
| 6 | accent | titles, headers, selection |
| 7 | foreground alias | near-white anchor |
| 8 | dim | zebra rows, separators, dividers |
| 9-15 | bright series ramp | multi-series charts |

The explicit background is what the screen paints; the explicit foreground
is what text uses. Slot 7 can legitimately equal the background in a shipped
palette, which is why renderers must use the explicit foreground.

## Contrast normalization

Every theme is normalized once after parsing, against its explicit
background. The engine measures WCAG contrast and lifts colors that fail
their floor, deterministically and hue-preserving.

| Role | Floor |
|---|---|
| foreground text | 4.5:1 |
| accent | 3.0:1 |
| dim | 3.0:1 |
| zebra-row text | 3.0:1 |
| series ramp and accents | 2.0:1 |

The dim floor and zebra-row floor share slot 8, so zebra-row text wins and
dim keeps the highest value that still clears it. Normalization happens in
memory only; shipped and user files are never rewritten. Renderers keep
reading the same palette accessors.

## Shipped themes

Twelve themes:

`x`, `berlin`, `bogota`, `helsinki`, `lahabana`, `london`, `madrid`,
`miami`, `oslo`, `paris`, `praha`, `tokio`.

- `x` is compiled in as the startup fallback.
- All twelve are embedded as seeding templates and written to the themes
  directory on first run.
- `helsinki` and `madrid` are light themes; `london` and `berlin` are
  grayscale-oriented.

## Loading order

1. The compiled-in `x` palette.
2. Theme files in the themes directory, seeded on first run.
3. A file reusing a shipped name replaces that theme in place.
4. New names are appended after the shipped set.

## Files and paths

```
~/.config/xtop/themes/*.jsonc              (Linux)
~/Library/Application Support/xtop/themes/ (macOS)
%APPDATA%\xtop\themes\                     (Windows)
```

## Switching themes

- `t` and `T` cycle forward and backward in the TUI.
- The command palette offers a Themes page.
- `xtop --ct <name>` persists the theme and running instances follow it
  within one tick.

## Authoring guidance

- Prefer the explicit background and foreground pair.
- Keep slots 9-15 as brighter siblings of slots 1-7 in the same order.
- Do not fight the normalizer; write the palette you want.
- Do not reorder roles.
- Test both a dark and a light terminal if the theme is intended for both.
