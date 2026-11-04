# Skill: build and test

Use this when you need to verify a repository without changing its scope.

## Kernel

```
cd ~/repos/xtop-cli/xtop
cargo build
cargo build --no-default-features
cargo build --features plugin-wasm,plugin-external
cargo test
./scripts/ci.sh
./scripts/audit.sh
```

## API contracts

```
cd ~/repos/xtop-cli/api
cargo check --workspace
cargo test --workspace
cargo doc --workspace --no-deps
./scripts/ci.sh
```

## Widgets

```
cd ~/repos/xtop-cli/widgets
cargo test --workspace
./scripts/ci.sh
```

## Layouts

```
cd ~/repos/xtop-cli/layouts
cargo test
./scripts/ci.sh
```

## Plugins

```
cd ~/repos/xtop-cli/plugins
./scripts/ci.sh
```

Optional toolchain-dependent stages:

```
./scripts/ci.sh wasm
./scripts/ci.sh external
./scripts/build-examples.sh --check
```

## Extensions

```
cd ~/repos/xtop-cli/extensions
cargo test --workspace
./scripts/ci.sh
```

## Effects

```
cd ~/repos/xtop-cli/effects
cargo test --workspace
./scripts/ci.sh
```

## Web

```
cd ~/repos/xtop-cli/web
npm ci
npm run build
```

## Interpretation rules

- A stage that was skipped is not a pass.
- A warning that the repository treats as an error is a failure.
- If a command cannot run because a toolchain is absent, say so explicitly
  and do not substitute a partial result.
- Record the command and the final result in your report.
