# api operations

## Local CI

```
cd ~/repos/xtop-cli/api
./scripts/ci.sh            # fmt, clippy, check, test
./scripts/ci.sh fmt
./scripts/ci.sh clippy
./scripts/ci.sh check
./scripts/ci.sh test
```

Stages:

- `fmt`: `cargo fmt --all -- --check`;
- `clippy`: `cargo clippy --workspace --all-targets -- -D warnings`;
- `check`: `cargo check --workspace`;
- `test`: `cargo test --workspace`.

No GitHub workflow exists in this repository. Local CI is the gate.

## Documentation build

```
cargo doc --workspace --no-deps
```

Must complete without warnings. The contract docs comment on host
obligations.

## Test coverage

The four crates carry tests for:

- capability gating and denied-capability messages;
- `top_processes` ordering and stable ties;
- the `AlertThresholds` serde round trip;
- glyph mapping tables (colors, borders, markers);
- a fake-frame effect test.

`extension-api` has no tests yet because it has no behavior beyond the
trait defaults.

## Grep gates

These must remain green:

- no `WidgetRegistration` outside `widget-api`;
- no `AlertThresholds` definition outside `plugin-api`;
- no `PluginWidgetFn` alias;
- no `plugins_dir_tmp` references;
- no Docker symbols.

## Adding a contract item

Checklist:

1. Identify the owning crate.
2. Prefer additive changes and default implementations.
3. Document semantics and host obligations.
4. Add a test if the item crosses a wire or persistence boundary.
5. Update `docs/` and `docs/changes.md` if behavior or compatibility
   changes.
6. Run local CI.
7. Update consumers with temporary path dependencies and verify.
8. Restore consumer manifests to floating git dependencies.
9. Update this context.

## Publish checklist (pending phase)

Not executable yet. When publication is approved:

1. Choose the release version and confirm the 0.1.0 policy change.
2. Publish in dependency order: `plugin-api`, `widget-api`, `effect-api`,
   `extension-api`.
3. Tag the repository.
4. Pin or switch every consumer to the published versions.
5. Remove floating git dependencies only after all consumers migrate.
6. Add per-repository release automation.
7. Record the decision in `docs/changes.md` and this context.

## Local multi-repo verification

Use a patch section or path dependencies, then:

```
cd ~/repos/xtop-cli/api && ./scripts/ci.sh
cd ~/repos/xtop-cli/widgets && ./scripts/ci.sh
cd ~/repos/xtop-cli/plugins && ./scripts/ci.sh
cd ~/repos/xtop-cli/extensions && ./scripts/ci.sh
cd ~/repos/xtop-cli/effects && ./scripts/ci.sh
cd ~/repos/xtop-cli/xtop && ./scripts/ci.sh
```

Restore the manifests before committing upstream.
