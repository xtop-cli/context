# Cross-repository workflows

Procedures that involve more than one repository. Per-repository procedures
live in the respective `operations.md` and in `resources/skills/`.

## 1. Set up the workspace

```
mkdir -p ~/repos/xtop-cli
cd ~/repos/xtop-cli
for repo in api effects extensions layouts plugins web widgets xtop; do
  [ -d "$repo" ] || git clone "https://github.com/xtop-cli/$repo.git" "$repo"
done
```

This context repository is cloned separately and placed at
`~/repos/xtop-cli/context` so that it sits next to the upstream checkouts
without being inside any of them.

## 2. Verify a single repository

Every Rust repository exposes the same local CI entry point:

```
cd ~/repos/xtop-cli/<repo>
./scripts/ci.sh
```

The kernel additionally has a structural audit:

```
cd ~/repos/xtop-cli/xtop
./scripts/audit.sh
```

The web repository uses npm:

```
cd ~/repos/xtop-cli/web
npm ci
npm run build
```

## 3. Develop across repositories with path dependencies

The shipped manifests point at remote git dependencies. To compile a
consumer against an uncommitted sibling checkout, temporarily replace the
git dependency with a path dependency or add a `[patch]` section.

Example for the kernel consuming a local `api` checkout:

```toml
[patch."https://github.com/xtop-cli/api"]
xtop-plugin-api = { path = "../api/crates/plugin-api" }
xtop-widget-api = { path = "../api/crates/widget-api" }
xtop-extension-api = { path = "../api/crates/extension-api" }
xtop-effect-api = { path = "../api/crates/effect-api" }
```

Or a direct path dependency in the consumer:

```toml
xtop-plugin-api = { path = "../api/crates/plugin-api" }
```

Rules:

- The temporary override is never committed.
- Before pushing, restore the floating git form and remove generated
  `Cargo.lock` files in library workspaces.
- A combined scratch workspace can wire every repository together for a
  single cross-ecosystem compile and test. Keep it outside the repositories
  or under a git-ignored directory.

## 4. Change a contract

Contract changes start in `api` and then propagate outward.

1. Read `api/architecture.md`, the affected contract document and
   `api/changes.md`.
2. Change the contract crate and update its tests and docs.
3. Run `cd ~/repos/xtop-cli/api && ./scripts/ci.sh`.
4. Validate consumers locally with temporary path dependencies.
5. Record the breaking change in `api/docs/changes.md`.
6. Update every consumer and its docs in the same effort.
7. Remove temporary overrides before pushing.
8. Update this context repository: `api/` documents and any affected
   per-repository document.

Order of propagation for a plugin-api change:

```
api -> xtop
    -> widgets
    -> plugins
    -> extensions (if it consumes plugin constants)
```

For a widget-api change:

```
api -> widgets -> xtop
```

For an extension-api or effect-api change:

```
api -> extensions or effects -> xtop
```

## 5. Add a widget

1. Read `widgets/authoring.md` and `api/widget-contract.md`.
2. Copy an existing crate or run `xtop widget scaffold <name>`.
3. Implement `render(f, state, area)` against `WidgetState`.
4. Add tests with the `testkit` feature of `xtop-widget-core`.
5. Register the widget name in the aggregator or pack table.
6. Add the crate to the workspace members and pack dependencies.
7. Run `cd ~/repos/xtop-cli/widgets && ./scripts/ci.sh`.
8. Install into the kernel with `xtop widget install <path>` for local use.
9. Document the widget and any recognized layout options in
   `widgets/widget-reference.md`.

Detailed procedure: `resources/skills/widget-authoring.md`.

## 6. Add a layout

1. Read `layouts/schema.md` and `layouts/authoring.md`.
2. Write a JSONC file with a unique `name` and a valid tree.
3. Follow the tiling rules: proportional stacks sum to 100, fixed lengths
   next to fills, or fills absorbing a sub-100 percentage remainder.
4. Validate with `xtop layout check <file>`.
5. For a personal layout, copy it into the platform layouts directory.
6. For a community layout, add it under `layouts/custom/`, open a pull
   request, and verify the embedded defaults order is untouched.
7. Run `cd ~/repos/xtop-cli/layouts && ./scripts/ci.sh`.
8. Update `layouts/defaults.md` if a default changed.

Detailed procedure: `resources/skills/layout-authoring.md`.

## 7. Add a native plugin

1. Read `api/plugin-contract.md` and `plugins/architecture.md`.
2. Scaffold with `xtop plugin scaffold <name>` (creates
   `plugins-dev/xtop-plugin-<name>/`) or create a crate in the plugins
   workspace.
3. Implement `Plugin`; declare only the capabilities actually used.
4. Keep `manifest().version` sourced from `env!("CARGO_PKG_VERSION")`.
5. Add tests; use a `HostState` double copied from the contract tests.
6. Add the crate to the plugins workspace and run
   `cd ~/repos/xtop-cli/plugins && ./scripts/ci.sh`.
7. Install into the kernel with `xtop plugin install <name>`.
8. Add the registration line under the feature guard in the kernel's
   `src/commands/share/bootstrap.rs`.
9. Document actions and permissions.

Detailed procedure: `resources/skills/plugin-authoring.md`.

## 8. Add an extension

1. Read `api/extension-contract.md` and `extensions/architecture.md`.
2. Implement `Extension`; `manifest()` declares server ids.
3. Implement `run_server`; drive the kernel only through
   `ExtensionContext::tick` and `execute_plugin`.
4. Keep domain behavior in plugins, not in the extension.
5. Add tests against a mock `ExtensionHost`.
6. Add the crate to the extensions workspace and run
   `cd ~/repos/xtop-cli/extensions && ./scripts/ci.sh`.
7. Wire the kernel command behind the appropriate feature flag.
8. Document the wire protocol and error mapping.

## 9. Add an effect

1. Read `effects/authoring.md` and `api/effect-contract.md`.
2. Implement `Effect` as a pure function of `(buffer, elapsed)` whenever
   possible.
3. Use wall-clock `elapsed`, never frame counts.
4. Become a no-op after the effect window.
5. Add tests with synthetic buffers and known durations.
6. Add the crate to the effects workspace and run
   `cd ~/repos/xtop-cli/effects && ./scripts/ci.sh`.
7. Wire the kernel behind the optional `effects` feature and the `effect`
   config key.
8. Add a catalog entry in `effects/catalog.md`.

## 10. Sync documentation into the website

`web/docs/` is an English mirror of the upstream Markdown. The script
expects the sibling checkouts:

```
cd ~/repos/xtop-cli/web
./scripts/sync-docs.sh
```

Then update the Spanish mirror under `i18n/es/docs/` for changed files.
The mirror is not generated automatically from English; translations are
maintained by hand.

After syncing:

```
cd ~/repos/xtop-cli/web
npm ci
npm run build
```

The prebuild step regenerates the search index.

## 11. Release and distribution

Release publishing is being introduced gradually. The kernel has a
tag-triggered GitHub Actions workflow; the other Rust repositories still
have no release automation. The current distribution paths are:

1. **Source install**: `install.sh` (Linux) and `install.ps1` (Windows).
2. **Cargo**: `cargo install --git https://github.com/xtop-cli/xtop
   --all-features`.
3. **AUR**: the `xtop-git` package tracks the `xtop` repository HEAD. A
   `-git` package does not require upstream tags, but its `PKGBUILD` must
   be refreshed when the source URL, build script or metadata changes.
4. **Website**: pushing to `main` triggers the Pages workflow.
5. **Prebuilt binaries**: pushing a `v*` tag triggers the kernel release
   workflow.

### Kernel release workflow

`.github/workflows/release.yml` runs on `v*` tags:

1. `gate` verifies the tag commit is on `main`, the tag matches
   `Cargo.toml`, and tests pass.
2. `build` produces `--all-features` binaries for linux gnu x86_64/aarch64,
   linux musl x86_64, macOS x86_64/aarch64 and Windows x86_64 MSVC.
3. `checksums`, `sign` and `release` validate the archives, write
   `SHA256SUMS`, sign it with keyless cosign, attest provenance and publish
   the GitHub Release.

Create the tag from a commit merged into `main`:

```
git checkout main && git pull --ff-only
git tag -a v0.1.0 -m "xtop v0.1.0"
git push origin v0.1.0
```

The `release` environment can be protected with required reviewers in
repository settings.

Release checklist: `resources/skills/release-core.md`.

## 12. Update this context

After any change that alters documented behavior:

1. Update the affected per-repository file.
2. Update `repositories.md` if workspace members or commands changed.
3. Update `current-state.md` if the status changed.
4. Update `llms.txt` if a new document was added or renamed.
5. Run a consistency check: search for stale crate names, widget names,
   counts, paths and versions.
6. Never store secrets, tokens or machine-specific paths.

Detailed procedure: `resources/skills/context-maintenance.md`.
