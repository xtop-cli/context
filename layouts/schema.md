# Layout schema

The layout format is implemented by the serde model and manual visitor in
`src/model.rs`, plus the comment stripper in `src/loader.rs`. There is no
separate JSON Schema file; this document mirrors the code.

## File conventions

- One layout per file.
- The display name is the `name` field, not the file name.
- Files use `.jsonc` or `.json`.
- Built-in defaults live in `layouts/default/` and are embedded in the crate.
- User layouts live in the platform config directory under `layouts/`.
- Names must be unique among defaults and user files; merging is exact and
  case-sensitive.

## Document structure

A layout document is a single object with exactly two keys:

```jsonc
{
  "name": "My Layout",
  "root": {
    "direction": "vertical",
    "areas": [
      { "widget": "header", "size": 3 },
      { "widget": "cpu", "size": "60%" },
      { "widget": "processes", "size": "*" }
    ]
  }
}
```

| Key | Type | Required |
|---|---|---|
| `name` | string | yes |
| `root` | area object | yes |

Any other top-level key is rejected. Inside the root and inside area objects,
extra keys are silently ignored.

The root uses the area grammar. Its `size` is parsed and validated but
discarded: the root always spans the whole render area.

## Area objects

```jsonc
{
  "size": <size>,
  "widget": "name",
  "options": { ... }
}
```

or

```jsonc
{
  "size": <size>,
  "direction": "vertical",
  "areas": [ ... ]
}
```

| Key | Type | Meaning |
|---|---|---|
| `size` | number, string, omitted | constraint from the parent split; omitted means fill |
| `widget` | string | widget id for a leaf |
| `options` | object | per-instance display options; only with `widget` |
| `direction` | string | `vertical` or `horizontal`, case-insensitive |
| `areas` | array | children of a split; missing or empty renders nothing |

Exactly one of `widget` and `direction` must be present. If both are
present, `widget` wins and the split keys are ignored. Extra area keys are
ignored.

### Options

`options` is optional; an absent key and an explicit `null` both mean default
behavior. It must be a JSON object and is stored verbatim as
`serde_json::Value`. The layout crate never interprets it; the kernel
forwards it to the widget renderer.

- Unknown keys inside `options` are preserved.
- Options on split nodes or on areas where `direction` wins are ignored.
- A widget without options serializes byte-identically to the pre-options
  format.
- The root area follows the same grammar.

## Size constraints

| JSONC spelling | Constraint | Effect |
|---|---|---|
| omitted | fill | remaining space |
| `3` | fixed length | 3 rows or columns |
| `"45%"` | percentage | 45 percent of the parent |
| `"*"` | fill | remaining space |

Parsing details:

- bare numbers must be integers in `0..=65535`; fractions and negatives
  fail;
- `"*"` is checked before the percentage branch;
- `"NN%"` trims all trailing `%` characters and parses the remainder as
  `u16`; the value is not range-checked against 100;
- any other string is an error.

Fill areas share leftover space equally.

## Direction

- `vertical`: children stack top to bottom, each spanning full width.
- `horizontal`: children sit left to right, each spanning full height.

Constraint translation in the kernel:

- fixed length to `Constraint::Length`;
- percentage to `Constraint::Percentage`;
- fill to `Constraint::Fill(1)`.

Percentages are relative to the full enclosing split. Fixed lengths are not
subtracted first.

## Tiling rules

A split leaves an orphan band when it has neither a fill sibling nor
percentage children summing to 100. Built-in presets follow these rules:

1. Pure percentage stacks: percentages sum to 100.
2. Fixed length plus fill: a fixed header or strip next to a fill sibling.
   Do not put percentages next to fixed lengths in the same split.
3. Fill alongside percentages: allowed when the percentages sum below 100;
   the fill absorbs the remainder.

A loader test verifies full tiling for every split of every detail preset.

## JSONC dialect

Comments are stripped before parsing; the result must be strict JSON.

- `//` line comments.
- `/* ... */` block comments, not nestable.
- The stripper is string-aware.
- Trailing commas are not supported.
- Use double quotes for values.

## Serialized shape

Re-serialization writes fixed lengths as numbers, percentages as strings,
fill as `"*"`, then either `widget` with optional `options`, or `direction`
with `areas`.

## Embedded defaults

The first seven embedded files are the mode-bound defaults in palette order.
The three `detail_*` presets follow at slots 7-9. Extras are appended after
the modes so mode indices never move.

| Slot | File | Name |
|---|---|---|
| 0 | `dashboard.jsonc` | `Dashboard` |
| 1 | `vertical.jsonc` | `Vertical` |
| 2 | `horizontal.jsonc` | `Horizontal` |
| 3 | `cpu_focus.jsonc` | `CPU Focus` |
| 4 | `memory_focus.jsonc` | `Memory Focus` |
| 5 | `network_focus.jsonc` | `Network Focus` |
| 6 | `process_focus.jsonc` | `Process Focus` |
| 7 | `detail_dashboard.jsonc` | `Detail Dashboard` |
| 8 | `detail_network.jsonc` | `Detail Network` |
| 9 | `detail_processes.jsonc` | `Detail Processes` |

Nine distinct widget ids are referenced across the defaults: `header`, `cpu`,
`memory`, `storage`, `network`, `processes`, `disk_io`, `summary`,
`sensors`.
