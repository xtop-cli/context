# Agent: renderers

## Scope

The `widgets` repository: widget crates, the shared engine, the base pack,
the blocks pack and community packs.

## Required reading

- `../../widgets/README.md`
- `../../widgets/architecture.md`
- `../../widgets/widget-reference.md`
- `../../widgets/authoring.md`
- `../../api/widget-contract.md`
- `../../conventions.md`

## Workflow

1. Decide whether the change belongs in a widget crate, the shared engine or
   a pack registry.
2. Render only from `WidgetState`; never import kernel types.
3. Use `xtop-widget-core` for formatting, charts, roles and frames.
4. Import glyph helpers from `xtop_widget_api::glyph`; never copy them.
5. Parse layout options defensively; ignore unknown keys.
6. Provide honest empty states for missing data.
7. Add tests with the `testkit` feature and the standard smoke sizes.
8. Run local CI.
9. Update the widget reference and the upstream docs.

## Guardrails

- No per-frame allocation of the whole snapshot.
- No wrapping of logical rows.
- No panics on tiny rectangles.
- No fabricated values.
- No private re-implementation of canonical glyph mappings.
- No kernel imports.

## Verification

```
cd ~/repos/xtop-cli/widgets
./scripts/ci.sh
```

Then install into a kernel checkout with `xtop widget install` and render
the widget at the standard sizes before declaring success.

## Output

Report the crate and pack touched, the options recognized, the sizes and
states tested, and the widget reference updates.
