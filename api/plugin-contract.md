# Plugin contract

`xtop-plugin-api` defines how the kernel hosts plugins and how plugins act
on the kernel. Two parties implement against it:

- the host (the kernel) implements `HostState`;
- plugins implement `Plugin` and act through `PluginContext`.

The crate depends only on `ratatui` (for the widget render closure) and
`serde` (for `AlertThresholds`).

## Public surface

| Item | Module | Notes |
|---|---|---|
| `Plugin` | `plugin.rs` | core trait |
| `PluginManifest` | `manifest.rs` | identity and declared capabilities |
| `PluginContext` | `context.rs` | capability-checked access to host state |
| `HostState` | `host.rs` | host-side view implemented by the kernel |
| `RuntimeConfig` | `host.rs` | theme, layout, interval, hostname |
| `AlertThresholds` | `host.rs` | serde contract type |
| `SystemDataProvider` | `provider.rs` | data source contract |
| `PluginCapability` | `capability.rs` | declared and enforced capabilities |
| `PluginError` | `error.rs` | recoverable, fatal, unknown action |
| `PluginWidget` | `widget.rs` | plugin widget registration over `HostState` |
| `hex_to_rgb` | `color.rs` | `#rrggbb` to `[u8; 3]` |
| `model` | `model.rs` | shared data model |

Everything except the `model` module is re-exported at the crate root.

## PluginManifest

```rust
pub struct PluginManifest {
    pub id: String,
    pub name: String,
    pub version: String,
    pub description: String,
    pub capabilities: Vec<PluginCapability>,
}
```

- `id` is stable and is used as the plugin data directory name.
- Consumers should source `version` from `env!("CARGO_PKG_VERSION")`.
- The kernel reads the manifest when registering the plugin.

## Plugin trait

```rust
pub trait Plugin: Debug + Send {
    fn manifest(&self) -> PluginManifest;

    fn on_enable(&mut self, _ctx: &mut PluginContext) -> Result<(), PluginError> { Ok(()) }
    fn on_disable(&mut self, _ctx: &mut PluginContext) -> Result<(), PluginError> { Ok(()) }
    fn on_tick(&mut self, _ctx: &mut PluginContext) -> Result<(), PluginError> { Ok(()) }
    fn on_key(&mut self, _ctx: &mut PluginContext, _key: &str) -> Result<bool, PluginError> { Ok(false) }
    fn data_provider(&self) -> Option<Box<dyn SystemDataProvider>> { None }
    fn widget(&self) -> Option<PluginWidget> { None }
    fn execute(&mut self, _ctx: &mut PluginContext, _action: &str, _params: &str)
        -> Result<String, PluginError> {
        Err(PluginError::UnknownAction(_action.to_string()))
    }
}
```

Lifecycle:

- `manifest` is the only required method.
- `on_enable` and `on_disable` run once each.
- `on_tick` runs every kernel tick. Plugins may throttle their own work.
- `on_key` returns `true` to consume the key.
- `data_provider` contributes extra metrics; the host collects it only from
  plugins that declared `ReadSystemInfo`.
- `widget` contributes one widget; the host collects it only from plugins
  that declared `RenderWidgets`.
- `execute` is the named action API used by agents, MCP and tools. It returns
  a JSON-like string.

The `Debug + Send` supertraits allow the host to box plugins and isolate
failures.

## Capability enforcement

`PluginContext` wraps a mutable host reference plus the plugin data
directory and the declared capabilities. Every state-touching method checks
the capability first. A denial is a `Recoverable` error with the message
`plugin does not have required capability: <capability>`.

| Method | Capability |
|---|---|
| `snapshot()` | `ReadSystemInfo` |
| `system_info()` | `ReadSystemInfo` |
| `top_processes(n)` | `ReadSystemInfo` |
| `kill_process(pid)` | `KillProcesses` |
| `set_alert_thresholds(cpu, mem, disk)` | `ModifyConfig` |
| `set_theme_by_name(name)` | `ModifyConfig` |
| `set_layout_by_name(name)` | `ModifyConfig` |
| `set_update_interval(ms)` | `ModifyConfig` |
| `alerts()` | none |
| `config()` | none |
| `data_dir()` | none |

Contract details:

- Reads return `Result` so the capability check can fail. Callers handle
  `Err`.
- `top_processes(n)` sorts by CPU descending with `total_cmp`, then
  truncates. The sort is stable: equal-CPU processes keep snapshot order.
- `data_dir()` is host-provided. The kernel creates it on registration by
  joining the base directory and the plugin id.
- `alerts()`, `config()` and `data_dir()` are ungated reads.
- `PluginContext::new(host, data_dir, capabilities)` is public; the kernel is
  the only caller today.

`PluginCapability` is `#[non_exhaustive]`, derives `Clone, Debug,
PartialEq`, and has variants `ReadSystemInfo`, `KillProcesses`,
`ModifyConfig`, `RenderWidgets` and `Custom(String)`.

Enforcement exists in two places:

1. `PluginContext` methods in this crate.
2. Host-side collection filters in the kernel's `PluginManager`, which only
   collects providers from `ReadSystemInfo` plugins and widgets from
   `RenderWidgets` plugins.

`Custom(String)` is an escape hatch with no built-in enforcement.

## HostState

The kernel implements this trait for its live application state. Plugins
never depend on kernel types.

Nine methods:

| Method | Meaning |
|---|---|
| `snapshot() -> SystemSnapshot` | full sample, possibly the cached one |
| `system_info() -> SystemInfo` | machine identity |
| `kill_process(pid) -> bool` | send termination; true if sent |
| `set_alert_thresholds(cpu, mem, disk)` | overwrite thresholds |
| `alerts() -> AlertThresholds` | current thresholds |
| `config() -> RuntimeConfig` | theme, layout, interval, hostname |
| `set_theme_by_name(name) -> bool` | true if the theme exists |
| `set_layout_by_name(name) -> bool` | true if the layout exists |
| `set_update_interval_ms(ms)` | tick interval |

Tests in `context.rs` implement a `FakeHost` over a hand-built snapshot.
Copy that pattern when writing a host double.

## PluginError

```rust
pub enum PluginError {
    Recoverable(String),
    Fatal(String),
    UnknownAction(String),
}
```

- `Recoverable` prints the message verbatim and the plugin keeps running.
- `Fatal` prints `FATAL: <message>` and the plugin should be disabled.
- `UnknownAction` prints `unknown action: <action>`.
- Implements `std::error::Error`.

The kernel's extension host maps plugin errors onto extension errors:
recoverable and fatal map one-to-one, unknown action becomes recoverable.

## SystemDataProvider

Source of system data. Implemented by the kernel provider and by plugins
that contribute metrics.

| Method | Default |
|---|---|
| `refresh_all(&mut self)` | required |
| `snapshot(&self) -> SystemSnapshot` | required |
| `disk_io() -> Vec<DiskIOInfo>` | empty |
| `batteries() -> Vec<BatteryInfo>` | empty |
| `gpu_info() -> Vec<GpuInfo>` | empty |
| `system_info() -> SystemInfo` | `SystemInfo::default()` |
| `kill_process(pid) -> bool` | false |
| `as_any()` / `as_any_mut()` | required |
| `add_extras(Vec<Box<dyn SystemDataProvider>>)` | no-op |

## PluginWidget

```rust
pub struct PluginWidget {
    pub name: String,
    pub render: Arc<dyn Fn(&mut ratatui::Frame, &dyn HostState, ratatui::Rect) + Send + Sync>,
}
```

A plugin widget renders over `&dyn HostState`. It is distinct from
`xtop_widget_api::WidgetRegistration`, which renders over `WidgetState`.
Plugin widgets take precedence over every widget pack.

## Authoring a plugin

1. Declare the dependency:

```toml
[dependencies]
xtop-plugin-api = { git = "https://github.com/xtop-cli/api" }
serde_json = "1"
```

2. Implement `Plugin`. Declare only the capabilities you use.
3. Implement `execute` with a documented action and params syntax.
4. Optional: return a `PluginWidget` from `widget()` or a provider from
   `data_provider()`.
5. Add tests using a `HostState` double.
6. Wire the plugin into the kernel behind a Cargo feature and register it in
   `src/commands/share/bootstrap.rs`.
7. Document the actions, params and capabilities.

The reference implementation is `xtop-plugin-samurai`.
