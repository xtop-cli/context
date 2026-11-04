# Samurai heuristic rules

The analyzer runs ten rules over the live process table and keeps up to 50
alerts per run. Each rule returns at most one alert per process per run.
Rules are evaluated in priority order and a process can produce several
alerts. Severities: Critical, Warning, Info.

All matching is substring or regex containment. There is no slash wrapping
and no whole-string anchoring.

## Rule 1 - Suspicious executable path

- Trigger: `exe_path` starts with `/tmp/`, `/dev/shm/`, `/var/tmp/`,
  `/proc/`, `/private/tmp/` or `/private/var/tmp/`.
- Severity: Critical.

## Rule 2 - Orphan process

- Trigger: `parent_pid == 1` and the lowercase process name does not contain
  an allowed daemon name (`systemd`, `init`, `launchd`, `sshd`, `login`,
  `getty`, `nginx`, `apache2`, `httpd`, `bash`, `sh`, `zsh`, `tmux`,
  `screen`).
- Severity: Critical when `run_time < 60` seconds, otherwise Warning.

## Rule 3 - Masquerading

Two checks:

1. The name is a known system process (`svchost`, `lsass`, `launchd`,
   `sshd`, `systemd`, `init`) but the executable is not under `/usr/`,
   `/bin/`, `/sbin/` or `/System/`: Critical.
2. Otherwise the name differs from the executable file stem
   (case-insensitive) and neither contains the other: Warning. Common
   aliases where the path contains the name are allowed.

## Rule 4 - Privilege escalation

- Trigger: `effective_user_id != user_id`, both present.
- Skips known SUID binaries: `sudo`, `passwd`, `ping`, `su`, `newgrp`,
  `gpasswd`, `chsh`, `chfn`, `mount`, `umount`.
- Severity: Critical when the effective user is root, otherwise Warning.

## Rule 5 - Suspicious browser child

- Trigger: the parent process is a browser (name contains `chrome`,
  `firefox`, `safari`, `edge`, `brave`, `opera` or `chromium`) and the
  child's name contains none of the expected helper words (`helper`,
  `plugin_container`, `plugin_host`, `gpu_process`, `renderer`, `utility`,
  `crashpad`, `updater`).
- Severity: Warning.

## Rule 6 - Known threat name or command

- Trigger: the lowercased name contains a known miner or rootkit name
  (`minerd`, `cpu_miner`, `xmrig`, `kdevtmpfsi`, `kinsing`, `diagree`,
  `watchbog`, `sysguard`, `crond64`, `mkfile`, `sysupdate`, `xmrig-nvidia`,
  `xmrig-amd`, `moneroocean`), or the command line matches a compiled
  pattern: miner flags `--donate-level`, `--max-cpu-usage`, `--threads`, or
  pool hosts `pool.monero`, `pool.supportxmr`, `mine.monero`.
- Severity: Critical.

## Rule 7 - Pipe or download command line

- Trigger: the command line matches any compiled pattern:
  - `curl ... | sh`, `| bash`, `| zsh`, and the wget equivalents;
  - a curl command ending in a bare `bash` or `sh`;
  - `python3 -c` with `import`, `urllib`, `requests` or `socket`;
  - `base64 -d |`;
  - `eval $(curl ...)`, `eval $(wget ...)`, and
    `bash -c ... $(curl|wget)`.
- Severity: Critical.

## Rule 8 - High thread count

- Trigger: `thread_count >= 500` and the name is not in the allow list
  (`chrome`, `firefox`, `code`, `Code`, `idea`, `java`, `dotnet`, `python`,
  `node`, `mysqld`, `postgres`, `Xorg`, `dockerd`).
- Severity: Critical when `thread_count > 1000` or `cpu_usage > 200.0`,
  otherwise Warning.

## Rule 9 - High file descriptor count

- Trigger: `open_files >= 1000` and the name is not in the allow list
  (`mysql`, `postgres`, `nginx`, `httpd`, `apache`, `chrome`, `firefox`,
  `code`, `java`, `dotnet`, `dockerd`).
- Severity: Info.

## Rule 10 - Spawn storm

- Trigger: more than five distinct fresh PIDs of the same process name
  within a rolling 120-second window. Fresh means `run_time <= 120` and the
  window is `now - start_time < 120`. A PID is counted once.
- Severity: Warning. The message reports how many new instances were seen.
- History is kept per plugin across runs.

## Notes for changes

- Thresholds and pattern lists are constants in `src/lib.rs`.
- Alert lists are replaced wholesale at the end of each run.
- During the first five ticks after start, no analysis has completed, so
  `process.alerts` and `alerts.status` are empty.
