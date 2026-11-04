# Agent: contracts

## Scope

The `api` repository: data model, plugin protocol, widget render contract,
extension protocol and effect protocol.

## Required reading

- `../../api/README.md`
- `../../api/architecture.md`
- `../../api/data-model.md`
- the relevant contract document
- `../../api/changes.md`
- `../../conventions.md`

## Workflow

1. Decide which crate owns the change using the DR-1 to DR-7 rules.
2. Prefer additive changes with default implementations.
3. Keep the data model free of serde unless the type is persisted or
   configured.
4. Keep capability-gated reads returning `Result`.
5. Keep the canonical glyph helpers as the only implementation.
6. Add tests for wire, persistence and capability behavior.
7. Document host obligations.
8. Run local CI.
9. Propagate to consumers with temporary path dependencies.
10. Record breaking changes in `../../api/changes.md` and update this
    context.

## Guardrails

- Never duplicate a contract type in a consumer.
- Never merge `PluginWidget` and `WidgetRegistration` into one type.
- Never add speculative registration or hook types without a consumer.
- Never add a dependency outside the policy.
- Never publish or tag without an explicit human decision.

## Verification

```
cd ~/repos/xtop-cli/api
./scripts/ci.sh
cargo doc --workspace --no-deps
```

## Output

Report the owning crate, the compatibility impact, the consumers verified,
the tests added and the migration path for breaking changes.
