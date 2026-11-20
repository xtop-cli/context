# Layout operations

## Local CI

```
cd ~/repos/xtop-cli/layouts
./scripts/ci.sh
./scripts/ci.sh fmt
./scripts/ci.sh clippy
./scripts/ci.sh check
./scripts/ci.sh test
```

No GitHub workflow exists in this repository.

## Adding a shipped layout

1. Add the JSONC file under `layouts/default/` (official) or
   `layouts/custom/` (community).
2. Only add it to `DEFAULT_LAYOUT_SOURCES` in `src/loader.rs` if it must be
   embedded in every installation. The base set is currently just
   Dashboard.
3. Add or update the catalog entry in the kernel `catalog.json` with id,
   display name, category, version, author, license, URL and SHA-256.
4. If the file is a mode layout, update the mode mapping and its tests.
5. Add or update tiling and parsing tests.
6. Update `docs/layout-schema.md`, `docs/authoring.md` and this context.

## Adding a community layout

1. Add the JSONC file under `layouts/custom/`.
2. Use a unique `name`.
3. Validate the structure.
4. Add or update `layouts/custom/README.md`.
5. Open a pull request.
6. Reviewers check tiling, widget ids and option names.

## Review checklist

- unique, case-sensitive name;
- valid JSONC and no trailing commas;
- only documented keys;
- full tiling for the common sizes;
- widget ids exist in a standard kernel build, or the layout documents the
  required plugin/pack;
- options are documented in the widgets repository;
- the catalog checksum matches the shipped file;
- CI passes.

## Kernel-side asset seeding

The kernel writes the embedded Dashboard into the user layouts directory on
first run or with `xtop --gen-config`. Existing files are never overwritten.
Every other layout is a catalog asset installed with
`xtop layout download <name>` and recorded in `assets.json`.

## Compatibility notes

- Layout files are portable across kernel builds.
- An unknown widget id is not a parse error.
- A layout with an empty split parses but renders nothing.
- Percentages above 100 are accepted syntactically and are meaningless in
  the renderer.
