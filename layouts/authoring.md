# Authoring layouts

## Where layouts live

| Platform | User layouts directory |
|---|---|
| Linux | `$XDG_CONFIG_HOME/xtop/layouts` or `~/.config/xtop/layouts` |
| macOS | `~/Library/Application Support/xtop/layouts` |
| Windows | `%APPDATA%\xtop\layouts` |

The kernel reads the directory at startup. Restart after adding a file.

## Writing a layout

```jsonc
{
  // My Layout: header, CPU chart and process list stacked
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

Rules of thumb:

- `name` must be unique. Same name replaces a default in place; new name is
  appended.
- Keep a 3-row header at the top, like the defaults.
- Use percentages inside a split, not next to fixed lengths.
- Comments with `//` and `/* */` are allowed; trailing commas are not.
- A widget node can carry `options`; the layout crate forwards them without
  interpretation.
- Validate before shipping.

## Density guidance

Dense layouts should cover every row and never starve a box.

- Pure percentage stacks tile exactly when they sum to 100.
- A fixed header or strip belongs next to a fill.
- A fill can absorb the remainder of percentages summing below 100.
- Avoid mixing percentages and fixed lengths in one split.

Comfortable box heights:

| Widget | Comfortable rows |
|---|---|
| `cpu` | 12-22 |
| `processes` | 9-30 |
| `summary` | 6-9 |
| `sensors` | 7-12 |
| `memory` | 6-10 |
| `network` | 6-26 |
| `disk_io` | 6-12 |
| `storage` | 5-9 |

Example at 100x34 (31 body rows):

- Detail Dashboard: 18-row monitor band over 13 rows of processes.
- Detail Network: 22-row network box over 9 rows of processes.
- Detail Processes: 8-row stat strip over a 23-row process table.

## Validate

```
xtop layout check my_layout.jsonc
```

It runs the same parser as the runtime and reports structural validity. It
cannot validate widget ids against a particular build.

Invalid files are skipped at load with a stderr message, so a broken file
does not crash the app; it simply does not appear.

## Activate

- Copy the file into the user layouts directory and restart.
- Cycle with `l`, or select it by name from the command palette.
- The active layout name is persisted in configuration.

Seeded templates are copies of the defaults. Editing a seeded file is the
supported way to customize a built-in; because overrides match by `name`, it
replaces the default in its slot. Existing files are never overwritten on
upgrade.

## Widget ids

| Id | Availability |
|---|---|
| `header`, `cpu`, `memory`, `storage`, `network`, `processes`, `disk_io` | base pack, always compiled |
| `summary`, `sensors` | base pack additions |
| `battery`, `gpu` | base pack, reachable through fullscreen |
| plugin and runtime widget names | when the plugin or runtime host is enabled |

An unknown id renders an empty area and warns once.

## Modes and thresholds

| Requested mode | Size | Effective layout |
|---|---|---|
| any | width under 60 or height under 14 | minimal |
| Dashboard | width under 80 | Vertical |
| Dashboard | width 80-99 or height under 28 | Compact |
| Dashboard | width 100+ and height 28+ | Dashboard |
| other modes | 60x14 or larger | requested mode |

Below 40x8 the UI shows a terminal-too-small notice before layout logic.

Mode slots 0-6 are fixed. Extras are appended and addressed by name. `l`
cycles through modes, then presets, then user layouts, then wraps.

## Share a community layout

1. Put `my_layout.jsonc` into `layouts/custom/`.
2. Use a unique name and validate it.
3. Open a pull request.
4. Users install with `xtop layout install my_layout`.

`xtop layout install` sparse-clones the repository, scans `layouts/custom/`,
matches by file stem or name, and copies the file into the user directory
without overwriting an existing one.

Users can also copy the file manually; community files are not read by the
kernel at runtime until installed or copied.

## Checklist

- unique name;
- valid JSONC, no trailing commas;
- full tiling under the documented rules;
- known widget ids;
- comfortable box heights;
- validated with `xtop layout check`;
- docs updated if a default or a recognized option changed.
