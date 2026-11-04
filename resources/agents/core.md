# Agent: kernel core

## Scope

The `xtop` repository: application behavior, CLI, configuration, state,
theme, UI render engine, overlays, providers, plugin hosting and extension
hosting.

## Required reading

- `../../xtop/README.md`
- `../../xtop/architecture.md`
- `../../xtop/operations.md`
- `../../conventions.md`
- the upstream module doc for the area being changed
- `../../api/plugin-contract.md` and `../../api/widget-contract.md` when the
  change crosses a contract boundary

## Workflow

1. Read the module map and the source of the area.
2. Confirm the current branch and worktree state.
3. Prefer changing the kernel only when the behavior is genuinely
   kernel-owned. Renderers belong in `widgets`; layouts in `layouts`;
   plugins in `plugins`; extensions in `extensions`; effects in `effects`;
   shared types in `api`.
4. Keep `cfg(target_os)` inside `providers/sysinfo/platform/`.
5. Keep files under 600 lines and avoid TODO/FIXME markers.
6. Guard for a missing snapshot and for tiny terminal areas.
7. Never fabricate hardware readings.
8. Run local CI and the audit.
9. Update kernel docs and this context when behavior changes.

## Guardrails

- Do not add a GitHub workflow.
- Do not add `tokio`, `clap` or `chrono`.
- Do not bypass plugin capability checks.
- Do not resample the system in the render path.
- Do not change the default layout slot order.
- Do not break the persisted config JSON keys.

## Verification

```
cd ~/repos/xtop-cli/xtop
./scripts/ci.sh
./scripts/audit.sh
cargo build --no-default-features
```

## Output

Report the modules touched, the behavior change, the commands run and any
remaining risk.
