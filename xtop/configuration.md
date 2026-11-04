# Configuration

The kernel persists configuration on quit. The file is `config.json` in the
platform config directory, next to `themes/` and `layouts/`.

## Locations

| Platform | Config directory |
|---|---|
| Linux | `$XDG_CONFIG_HOME/xtop` or `~/.config/xtop` |
| macOS | `~/Library/Application Support/xtop` |
| Windows | `%APPDATA%\xtop` |

Files:

```
config.json
themes/*.jsonc
layouts/*.jsonc
plugins/<plugin-id>/
wasm/*.wasm            (runtime widgets)
external/*.json        (runtime widgets)
```

## Schema

| Key | Type | Default | Meaning |
|---|---|---|---|
| `theme` | string | `"x"` | active theme name |
| `layout_mode` | string | `"Dashboard"` | built-in mode |
| `layout_name` | string | `""` | active custom or preset layout name; takes precedence over `layout_mode` when it resolves |
| `update_interval_ms` | integer | `1000` | sampling interval, clamped to 100 through 3,600,000 |
| `history_points` | integer | `100` | retained chart points |
| `alerts` | object | 90/90/90 | alert thresholds |
| `keybindings` | object | see below | action to key list |
| `style` | object | classic | chart charset, borders, pack selection |
| `effect` | string or absent | absent | `"fade"` activates the effect when compiled with `effects` |

`layout_mode` values: `Dashboard`, `Vertical`, `Horizontal`, `CpuFocus`,
`MemoryFocus`, `NetworkFocus`, `ProcessFocus`.

Unknown keys and missing optional keys are ignored. A file that cannot be
parsed falls back to defaults.

## Example

```json
{
  "theme": "miami",
  "layout_mode": "Dashboard",
  "layout_name": "",
  "update_interval_ms": 1000,
  "history_points": 100,
  "alerts": {
    "cpu_high": 90.0,
    "mem_high": 90.0,
    "disk_high": 90.0
  },
  "keybindings": {
    "quit": ["q"],
    "help": ["?"],
    "next_theme": ["t"],
    "prev_theme": ["T"],
    "next_layout": ["l"],
    "toggle_fullscreen": ["f"],
    "cycle_fullscreen": ["F"],
    "search": ["/"],
    "command_palette": ["ctrl+p", "ctrl+P"],
    "cancel": ["escape"],
    "kill_process": ["k"],
    "process_up": ["up"],
    "process_down": ["down"],
    "cycle_sort": ["s"]
  },
  "style": {
    "charset": "braille",
    "borders": "native",
    "pack": null,
    "widgets": {}
  }
}
```

## Alerts

| Key | Default | Effect |
|---|---|---|
| `cpu_high` | 90.0 | CPU warning threshold |
| `mem_high` | 90.0 | memory warning threshold |
| `disk_high` | 90.0 | disk warning threshold |

Exceeding a threshold changes the widget color and adds a warning marker to
its title.

## Keybindings

Each action accepts a list of key strings; the first match wins.

Supported spellings:

- single characters: `"q"`, `"/"`, `"?"`;
- shifted characters: `"T"`, `"F"`;
- modified keys: `"ctrl+p"`, `"alt+x"`;
- named keys: `escape`, `enter`, `backspace`, `tab`, `up`, `down`, `left`,
  `right`, `delete`, `home`, `end`, `pageup`, `pagedown`.

Actions: `quit`, `help`, `next_theme`, `prev_theme`, `next_layout`,
`toggle_fullscreen`, `cycle_fullscreen`, `search`, `command_palette`,
`cancel`, `kill_process`, `process_up`, `process_down`, `cycle_sort`.

`ctrl+p` also works as a hardcoded fallback for the command palette, which
covers terminals that report the modifier with different casing.

## Style

Values come from the shared widget-api enums.

- `charset`: `braille` (default), `dot`, `block`, `half_block`, `bar`.
- `borders`: `native` (default), `rounded`, `double`, `plain`, `ascii`.
  `plain` and `ascii` both draw the pure `+ - |` frame.
- `pack`: pack used for every widget without a per-widget override.
- `widgets`: per-widget overrides, each accepting `charset`, `borders` and
  `pack`.

Per-widget overrides beat global values. Layout-node `options` beat both for
the widget instance.

## Save behavior

On quit, the kernel reloads the existing file and overwrites only the fields
the runtime owns:

- `theme`;
- `layout_name` and `layout_mode`;
- `update_interval_ms`;
- `alerts`;
- `keybindings`.

User values such as `history_points` and `style` are preserved.

## Asset seeding

On startup, before the TUI or asset-dependent commands, the kernel seeds
templates into the config directory:

- 12 theme templates from `assets/themes/`;
- 10 layout templates from the layouts crate.

Seeding is guarded by a marker file `.xtop_initialized` whose content is the
asset version. At the current snapshot the version is `5`.

Rules:

- a file is written only when missing;
- user edits are never overwritten;
- a version bump re-seeds only missing files;
- renamed templates leave older copies in place, and those older copies
  behave as user layouts if their name no longer matches a built-in.

## Effect selection

`effect` is optional. `"fade"` activates the 500 ms fade-in only in builds
compiled with the `effects` feature. Any other value, or a missing key,
disables effects. Builds without the feature ignore the key.
