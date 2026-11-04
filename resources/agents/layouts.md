# Agent: layouts

## Scope

The `layouts` repository: layout model, JSONC parser, merging, modes, the ten
embedded defaults and community layouts.

## Required reading

- `../../layouts/README.md`
- `../../layouts/schema.md`
- `../../layouts/authoring.md`
- `../../layouts/defaults.md`
- `../../conventions.md`

## Workflow

1. Choose between a user layout, a community layout and an embedded default.
2. Write or edit JSONC with strict syntax.
3. Apply the tiling rules.
4. Validate with `xtop layout check` when a kernel build is available, and
   with the crate tests otherwise.
5. Never renumber the first seven embedded slots.
6. Append new embedded extras at the end.
7. Add or update tests that pin order and tiling.
8. Run local CI.
9. Update the schema docs, the defaults table and this context.

## Guardrails

- No trailing commas.
- No undocumented top-level keys.
- No widget-id validation at parse time; ids are runtime strings.
- No percentage above 100 with a meaningful expectation.
- No change to name-based merging semantics without a recorded decision.

## Verification

```
cd ~/repos/xtop-cli/layouts
./scripts/ci.sh
```

## Output

Report the layout names and files touched, the tiling rules applied, the
validation performed and the documents updated.
