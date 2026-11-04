# Skill: plugin authoring

## 1. Prepare

```
cd ~/repos/xtop-cli/plugins
git status
```

Decide whether the plugin belongs in this workspace or in an external
repository.

## 2. Create the crate

```
plugins/xtop-plugin-<name>/
```

Manifest:

```toml
[package]
name = "xtop-plugin-<name>"
version = "0.1.0"
edition = "2021"

[dependencies]
xtop-plugin-api = { git = "https://github.com/xtop-cli/api" }
serde_json = "1"
```

## 3. Implement

- `manifest()` with a stable id, name, crate version and declared
  capabilities.
- `on_enable`, `on_tick`, `on_key`, `execute` as needed.
- Optional `data_provider()` and `widget()`.
- Only declare capabilities you actually use.

## 4. Params and responses

- `execute` receives plain strings.
- Define a small, documented params syntax.
- Return JSON strings.
- Return `Recoverable` for bad params, `UnknownAction` for unknown actions.

## 5. Tests

- Build a `HostState` double from the contract tests.
- Test capability denial paths.
- Test each action with valid and invalid params.
- Test the widget render if provided.

## 6. Register in the workspace

- add the crate to `members`;
- add any shared dependencies to the workspace table;
- run `./scripts/ci.sh`.

## 7. Wire into the kernel

From a kernel checkout:

```
xtop plugin install <name>
```

or add the dependency and feature manually:

```toml
[dependencies]
xtop-plugin-<name> = { git = "https://github.com/<owner>/<repo>", optional = true }

[features]
plugin-<name> = ["dep:xtop-plugin-<name>"]
```

Then add the registration line in `src/commands/share/bootstrap.rs` under
`#[cfg(feature = "plugin-<name>")]`.

## 8. Enable

Add the feature to the kernel default list or build with
`--features plugin-<name>`.

## 9. Document

- plugin README;
- action table;
- capabilities;
- data directory usage;
- widget name if any;
- this context `plugins/`.

## Checklist

- [ ] stable id
- [ ] version from crate
- [ ] minimal capabilities
- [ ] tests
- [ ] workspace member
- [ ] kernel feature and registration
- [ ] docs
- [ ] CI passes
