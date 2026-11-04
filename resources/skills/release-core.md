# Skill: kernel release verification

This is a verification procedure. It does not create tags, publish crates or
invent a release process.

## 1. Record the baseline

```
for repo in api effects extensions layouts plugins web widgets xtop; do
  printf '%s ' "$repo"
  git -C ~/repos/xtop-cli/$repo rev-parse --short HEAD
  git -C ~/repos/xtop-cli/$repo status --short
done
```

## 2. Run local CI

```
cd ~/repos/xtop-cli/api        && ./scripts/ci.sh
cd ~/repos/xtop-cli/widgets    && ./scripts/ci.sh
cd ~/repos/xtop-cli/layouts    && ./scripts/ci.sh
cd ~/repos/xtop-cli/plugins    && ./scripts/ci.sh
cd ~/repos/xtop-cli/extensions && ./scripts/ci.sh
cd ~/repos/xtop-cli/effects    && ./scripts/ci.sh
cd ~/repos/xtop-cli/xtop       && ./scripts/ci.sh
```

## 3. Audit the kernel

```
cd ~/repos/xtop-cli/xtop
./scripts/audit.sh
```

## 4. Build variants

```
cargo build --release
cargo build --release --no-default-features
cargo build --release --features plugin-wasm,plugin-external
cargo build --release --all-features
```

## 5. Verify installers

- `install.sh --check-deps`
- `install.sh --help`
- `install.ps1` syntax review (Windows execution not assumed)

Do not run an installer that writes to `/usr/local/bin` unless the
environment allows it.

## 6. AUR package

If the package is in scope:

1. Update `pkgver` to the current `r<commits>.<hash>`.
2. Regenerate `.SRCINFO`.
3. Run `makepkg -f` in the AUR checkout.
4. Verify the package contains the binary, license and README.
5. Commit with `upgpkg: xtop-git <pkgver>-<pkgrel>`.
6. Push only when explicitly asked.

## 7. Documentation

- Sync upstream docs into `web`.
- Update the Spanish mirror.
- Rebuild the site.
- Verify the Pages workflow inputs.

## 8. Context

Update `current-state.md`, `repositories.md` and any behavior document that
changed.

## 9. Report

For each step, state pass, fail or skipped with the exact reason. Do not
describe skipped work as complete.
