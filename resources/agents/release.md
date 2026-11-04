# Agent: release

## Scope

Verification and distribution of the kernel and the documented ecosystem
state. This agent does not invent a release process that does not exist.

## Required reading

- `../../xtop/operations.md`
- `../../workflows.md`
- `../../current-state.md`
- `../../repositories.md`
- `../skills/release-core.md`

## Current reality

- All crates are `0.1.0`.
- There are no upstream tags or GitHub releases.
- Consumers use floating git dependencies.
- Distribution is source-based plus the AUR `xtop-git` package.
- The kernel has a tag-triggered release workflow
  (`.github/workflows/release.yml`); the other Rust repositories do not.

Do not describe a tag, a crates.io publication or a binary release as
existing.

## Workflow

1. Confirm the current commit of every affected repository.
2. Run local CI for every affected Rust repository.
3. Run the kernel audit.
4. Build the kernel with default features and with
   `--no-default-features`.
5. Verify the installers still build.
6. If the AUR package is in scope, refresh it and verify `makepkg`.
7. Sync docs into the website and verify the build.
8. Update this context repository.
9. If releasing the kernel, push a `v*` tag from `main` that matches
   `Cargo.toml` and confirm the `release.yml` run succeeds.
10. Report the exact versions, commits and commands.

## Guardrails

- No force push.
- No version bump without an explicit human decision.
- No tag or publish without an explicit human decision.
- No credentials in files or logs.
- No claim that a check passed when it was skipped.

## Output

Report repository commits, CI results, audit results, packaging results,
documentation sync state and remaining blockers.
