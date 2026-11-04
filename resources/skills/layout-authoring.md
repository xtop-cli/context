# Skill: layout authoring

## 1. Decide the target

- personal layout: user config directory;
- community layout: `layouts/custom/` plus a pull request;
- embedded default: only when every installation should ship it.

## 2. Write the file

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

## 3. Apply tiling rules

- percentages in a split sum to 100, or;
- a fixed length sits next to a fill, or;
- percentages sum below 100 next to a fill.

Do not mix fixed lengths and percentages in one split.

## 4. Use known widget ids

Base pack: `header`, `cpu`, `memory`, `storage`, `network`, `processes`,
`disk_io`, `battery`, `gpu`, `summary`, `sensors`.

Plugin and runtime widget names are valid when the corresponding feature is
enabled.

## 5. Add options only when documented

Recognized keys are listed in `../../widgets/widget-reference.md`. Unknown
keys are ignored.

## 6. Validate

```
xtop layout check my_layout.jsonc
```

Without a kernel build, run the layout crate tests, which include parsing
and tiling checks.

## 7. Activate locally

Copy the file into the user layouts directory and restart:

| Platform | Directory |
|---|---|
| Linux | `$XDG_CONFIG_HOME/xtop/layouts` or `~/.config/xtop/layouts` |
| macOS | `~/Library/Application Support/xtop/layouts` |
| Windows | `%APPDATA%\xtop\layouts` |

## 8. Publish a community layout

1. Add the file under `layouts/custom/`.
2. Use a unique name.
3. Update `layouts/custom/README.md` if it lists layouts.
4. Run `cd ~/repos/xtop-cli/layouts && ./scripts/ci.sh`.
5. Open a pull request.
6. Users install with `xtop layout install <name>`.

## 9. Embedded defaults

1. Append the file to `layouts/default/`.
2. Append to `DEFAULT_LAYOUT_SOURCES`.
3. Never renumber the first seven slots.
4. Add tiling and registry tests.
5. Bump the kernel asset seed version.
6. Update `layouts/defaults.md` and upstream docs.

## Checklist

- [ ] unique name
- [ ] valid JSONC, no trailing commas
- [ ] full tiling
- [ ] known widget ids
- [ ] documented options
- [ ] validated
- [ ] CI passes
- [ ] docs and context updated
