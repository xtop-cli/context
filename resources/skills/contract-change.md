# Skill: contract change

Use this when a type, trait or protocol that crosses a repository boundary
must change.

## 1. Classify

- Is it in the data model or plugin protocol? `plugin-api`.
- Widget render contract? `widget-api`.
- Extension protocol? `extension-api`.
- Effect protocol? `effect-api`.
- Layout format? `layouts`, not `api`.
- Ecosystem constant? The producer repository, not `api`.

## 2. Design

- Prefer additive changes.
- Add default methods where possible.
- Keep capability-gated reads returning `Result`.
- Do not add a type before a real consumer exists.
- Do not merge distinct contract types that happen to look alike.

## 3. Implement in api

```
cd ~/repos/xtop-cli/api
```

- change the crate;
- update crate docs;
- add or update tests;
- update `docs/<contract>.md`;
- record breaking changes in `docs/changes.md`;
- run `./scripts/ci.sh`.

## 4. Validate consumers locally

Use a patch section in each consumer:

```toml
[patch."https://github.com/xtop-cli/api"]
xtop-plugin-api = { path = "../api/crates/plugin-api" }
```

or a direct path dependency.

Run CI in this order:

```
api -> widgets -> plugins -> extensions -> effects -> xtop
```

Skip repositories the change cannot affect.

## 5. Propagate

Update every affected consumer in the same effort. If the change is
breaking, update the migration notes at the same time.

## 6. Restore

- Remove every path dependency and patch section.
- Remove lockfiles generated in library workspaces.
- Confirm manifests match the floating git form.
- Re-run CI on at least one clean consumer to confirm it resolves the remote
  dependency.

## 7. Update context

- `api/` docs;
- affected per-repository docs;
- `conventions.md` if the rule changed;
- `current-state.md` if compatibility or status changed.

## 8. Report

State the owning crate, the compatibility impact, the consumers verified and
the migration path.
