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

## Adding an embedded default

Only do this when the layout belongs in every installation.

1. Add the file under `layouts/default/`.
2. Append it to `DEFAULT_LAYOUT_SOURCES` in `src/loader.rs`.
3. Never insert before the first seven slots.
4. If the file is a mode layout, update the mode mapping and its tests.
5. Add or update tiling tests.
6. Update `docs/layout-schema.md`, `docs/authoring.md` and this context.
7. Bump the kernel asset seed version so existing installs receive the new
   template.

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
- no change to the first seven embedded slots;
- CI passes.

## Kernel-side asset seeding

The kernel writes the embedded defaults into the user layouts directory on
first run and after an asset version bump. Existing files are never
overwritten. When a template is renamed, older copies remain and behave as
user layouts if their name no longer matches a built-in.

## Compatibility notes

- Layout files are portable across kernel builds.
- An unknown widget id is not a parse error.
- A layout with an empty split parses but renders nothing.
- Percentages above 100 are accepted syntactically and are meaningless in
  the renderer.
