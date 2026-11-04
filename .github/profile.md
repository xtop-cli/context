# xtop-cli

A cross-platform TUI system monitor written in Rust, built as a thin kernel
around a modular ecosystem.

- `xtop` - the application: ratatui interface, sysinfo providers, CLI and
  plugin hosting.
- `api` - shared contract crates: plugin, widget, extension and effect
  protocols plus the system data model.
- `widgets` - widget packs: one crate per renderer, plus a shared engine.
- `layouts` - data-driven JSONC layouts, seven modes and three detail
  presets.
- `plugins` - samurai monitoring plugin, sandboxed WASM widget host and
  external helper-process host.
- `extensions` - server-style integrations, starting with an MCP server.
- `effects` - frame effects over the rendered buffer.
- `web` - the landing page and documentation site.

Every repository compiles standalone against the contracts in `api` and
never against the kernel. A plain kernel build requires no optional
ecosystem piece.

Start with the kernel documentation at
`https://github.com/xtop-cli/xtop` and the site at
`https://xtop-cli.github.io/web/`.

License: MIT.
