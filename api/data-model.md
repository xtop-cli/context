# Shared data model

The system data model lives in `xtop_plugin_api::model`
(`crates/plugin-api/src/model.rs`). It is the single definition of the
metric types used by the kernel, widgets, plugins and extensions.

## Conventions

- All model structs derive `Debug, Clone`.
- Only `SystemInfo` derives `Default`.
- No model type derives `serde`. Model values cross repository boundaries as
  plain Rust values.
- Serde is reserved for persisted or configured types:
  `AlertThresholds` and the glyph enums in `xtop-widget-api`.

## Struct inventory

| Struct | Meaning |
|---|---|
| `SystemSnapshot` | one sample of the whole machine, assembled per tick |
| `CpuInfo` | one logical CPU core |
| `MemoryInfo` | physical memory totals, usage and percent |
| `SwapInfo` | swap totals, usage and percent |
| `DiskInfo` | one mounted filesystem |
| `DiskIOInfo` | cumulative and current throughput for a device |
| `NetworkInfo` | cumulative and current throughput plus IPs for an interface |
| `ProcessInfo` | one process with a 22-field forensic set |
| `LoadAvg` | 1/5/15-minute load averages |
| `BatteryInfo` | one battery |
| `GpuInfo` | one GPU |
| `SystemInfo` | slowly changing machine identity |

`SystemSnapshot` fields:

```
cpus, memory, swap, disks, networks, processes,
load_avg, uptime, cpu_temp, disk_io, batteries, gpus, sys_info
```

## Provider population

The kernel's `SysinfoProvider` populates the snapshot. Platform probes under
`providers/sysinfo/platform/` provide the extras.

| Struct | Source |
|---|---|
| `CpuInfo` | sysinfo per-core usage; `cpu_id` is the enumeration index; `frequency` in MHz; `governor` from the platform probe; `temp_c` from the Linux coretemp probe when the sensor mapping is unambiguous |
| `MemoryInfo` | sysinfo totals and usage; `percent = used / total * 100`, `0.0` when total is zero |
| `SwapInfo` | sysinfo swap values; same percent rule |
| `DiskInfo` | sysinfo disks; `used = total - available`; mount options from the platform probe |
| `DiskIOInfo` | sysinfo disk usage; speeds are byte deltas since the previous refresh over elapsed time |
| `NetworkInfo` | sysinfo networks; speeds from deltas; IPs from the platform probe |
| `ProcessInfo` | sysinfo process table plus platform probes for thread counts |
| `LoadAvg` | `System::load_average()` |
| `BatteryInfo` | Linux sysfs, macOS `pmset`, Windows `GetSystemPowerStatus`; empty elsewhere |
| `GpuInfo` | shared nvidia-smi probe, then Linux sysfs fallback when empty |
| `SystemInfo` | captured once at provider construction except package power |
| `cpu_temp` | maximum over sysinfo components |
| `uptime` | `System::uptime()` in seconds |

`SystemInfo` fields: `hostname`, `os_version`, `kernel`, `desktop_env`,
`shell`, `cpu_model`, `package_power_w`.

- Identity fields are captured once at construction.
- `cpu_model` is the sysinfo brand string of the first logical core; an
  empty brand yields `None`.
- `package_power_w` is sampled every refresh from Intel RAPL on Linux and is
  `None` where no readable source exists. The first sample after boot is
  `None` because there is no previous counter. JSON, DMI, hwmon or other
  counter sources are not fabricated.

## ProcessInfo field groups

22 fields:

- Base (7): `pid`, `name`, `cpu_usage`, `memory`, `user_id`, `state`, `cmd`
  (first argv element).
- P0 identity (3): `exe_path`, `parent_pid`, `cmd_full`.
- P1 forensics (6): `start_time` (epoch seconds), `run_time`,
  `effective_user_id`, `group_id`, `cwd`, `thread_count`.
- P2 resources (6): `open_files`, `open_files_limit`,
  `disk_total_read_bytes`, `disk_total_write_bytes`, `environ`,
  `session_id`.

Ordering rules applied to `snapshot().processes`:

1. The provider sorts by `cpu_usage` descending and truncates to a cap
   (`DEFAULT_MAX_PROCESSES = 200`, overridable with
   `XTOP_MAX_PROCESSES`).
2. `PluginContext::top_processes(n)` re-sorts and truncates itself, so the
   plugin-facing "top n by CPU" guarantee does not depend on producer
   ordering.

The uid-to-login-name mapping is not part of the model. It is a display
mapping exposed by `WidgetState::uid_to_name(uid)`. The kernel resolves it
from `/etc/passwd` on unix, Directory Services on macOS and local accounts
on Windows; renderers fall back to the numeric uid. Per-process CPU history
is also a widget-view concern, exposed by
`WidgetState::process_cpu_history(pid)`.

## Snapshot lifecycle

1. The kernel ticks at `update_interval_ms` (default 1000 ms).
2. `refresh_all()` refreshes sysinfo, disks, networks and components, and
   records counters as baselines for the next rate computation.
3. `snapshot()` assembles one `SystemSnapshot`.
4. `AppState` caches the sample. Every widget and action in that frame reads
   the same cache. The render path never resamples.
5. History buffers are pushed from the same sample.
6. Plugin tick handlers run against a context that returns the same
   snapshot family.

Before the first tick, `WidgetState::snapshot()` returns `None`.
`HostState::snapshot()` forces one fresh sample when the cache is empty.

## Removed Docker surface

The contract removed:

- the `DockerInfo` struct;
- the `SystemSnapshot::dockers` field;
- the `SystemDataProvider::docker_info` default method and re-export.

Nothing consumed them; the kernel always assigned an empty vector. A grep
for Docker symbols in `api` is empty. Docker metrics are not planned until a
real consumer exists.

## AlertThresholds serde contract

```rust
pub struct AlertThresholds {
    pub cpu_high: f64,
    pub mem_high: f64,
    pub disk_high: f64,
}
```

- derives `Debug, Clone, Serialize, Deserialize`;
- no `Default`, `Copy` or `PartialEq` on the contract type;
- no rename attributes; JSON keys are exactly `cpu_high`, `mem_high`,
  `disk_high`;
- the kernel keeps config-level defaults (90/90/90) next to its persisted
  schema.

Pinned serialized form:

```json
{"cpu_high":90.0,"mem_high":85.5,"disk_high":88.0}
```
