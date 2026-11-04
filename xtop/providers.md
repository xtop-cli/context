# Data providers

The kernel samples the system through `SysinfoProvider` and composes extra
providers from plugins through `CompositeProvider`.

## SysinfoProvider

`src/providers/sysinfo/provider.rs` populates the shared data model:

- CPU usage, frequency and per-core identity from sysinfo.
- Governor and per-core temperature from platform probes.
- Memory and swap totals and usage.
- Disks, mount points and mount options.
- Disk I/O counters and derived rates.
- Network interfaces, counters, derived rates and IP addresses.
- Process table and platform-specific thread counts.
- Load average and uptime.
- Batteries and GPUs from platform probes.
- System identity captured once at construction.
- Package power from Intel RAPL on Linux, sampled every refresh.

Rate computation records byte counters as baselines and compares them over
elapsed time on the next refresh.

## Process cap

The provider sorts processes by CPU descending and truncates to a bound:

- default 200;
- override with `XTOP_MAX_PROCESSES`.

The plugin contract independently sorts and truncates in
`top_processes(n)`.

## System identity

Captured once:

- hostname;
- OS version;
- kernel version;
- desktop environment from `XDG_CURRENT_DESKTOP` or `DESKTOP_SESSION`;
- shell from `SHELL` or `ComSpec`;
- CPU model from the sysinfo brand string of the first logical core.

Sampled every refresh:

- package power in watts from Intel RAPL energy-counter deltas, with
  wrap-around-safe deltas and a first-sample baseline of `None`.

The value is never fabricated. An unreadable source yields `None` and resets
the baseline.

## CompositeProvider

Wraps a primary provider and a list of extras.

| Method | Rule |
|---|---|
| `refresh_all` | refresh primary and every extra |
| `snapshot` | delegate to primary |
| `disk_io` | primary if non-empty, else first non-empty extra |
| `batteries` | primary if non-empty, else first non-empty extra |
| `gpu_info` | primary if non-empty, else first non-empty extra |
| `system_info` | primary if hostname is non-empty, else first non-empty extra |
| `kill_process` | primary first, then extras |
| `add_extras` | replace the extras list |

`SystemDataProvider` defaults return empty vectors or `SystemInfo::default()`
so a plugin only overrides what it actually provides.

## Platform coverage

| Signal | Linux | macOS | Windows | Fallback |
|---|---|---|---|---|
| CPU usage / frequency | yes | yes | yes | yes |
| Per-core temperature | coretemp when mapping is unambiguous | none per core | none per core | none |
| Aggregate temperature | sysinfo components | sysinfo components | thermal zones via sysinfo | none |
| Memory / swap | yes | yes | yes | yes |
| Disks | sysinfo | sysinfo | sysinfo | partial |
| Mount options | `/proc` and sysfs probes | `mount` | volume APIs | empty |
| Network interfaces and IPs | sysfs and `/proc` | `getifaddrs` | `GetAdaptersAddresses` | partial |
| Thread counts | `/proc` | `proc_pidinfo` | toolhelp snapshots | empty |
| User names | `/etc/passwd` | Directory Services | local accounts | numeric uid |
| Batteries | `/sys/class/power_supply` | `pmset` | `GetSystemPowerStatus` (aggregate) | empty |
| GPU | nvidia-smi plus `/sys/class/drm` | nvidia-smi | nvidia-smi | empty |
| Package power | Intel RAPL | none | none | none |

## Honest data rules

- Missing sensor, battery or GPU data produces an empty state, never a
  synthetic value.
- GPU utilization for AMD/Intel on macOS and Windows is not available
  through a public API and is not fabricated.
- Per-core temperatures are only exposed where the sensor mapping is
  unambiguous.
- The first package-power sample after boot is `None` by design.

## Adding a platform probe

1. Add the module under `src/providers/sysinfo/platform/<os>/`.
2. Keep `cfg(target_os)` inside that tree; the audit fails otherwise.
3. Return empty values on unsupported paths.
4. Wire the probe into the shared provider.
5. Add tests for parsing and fallback behavior.
6. Update `context/xtop/providers.md` and `api/data-model.md` if the data
   contract changes.
