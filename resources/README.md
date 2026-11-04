# Resources for automated work

This folder packages the context repository for AI coding agents and for
humans who want a repeatable procedure.

## Agents

Agents are role definitions. Each file states the scope, the required
reading, the workflow, the guardrails and the expected output for one kind
of work.

| Agent | Scope |
|---|---|
| `agents/core.md` | kernel application work |
| `agents/contracts.md` | contract changes in `api` |
| `agents/renderers.md` | widget packs and renderers |
| `agents/layouts.md` | layout model, schema and shipped layouts |
| `agents/functionality.md` | plugins, runtime widgets, extensions and effects |
| `agents/web.md` | site, documentation mirror and translations |
| `agents/release.md` | distribution, packaging and release verification |
| `agents/context-maintenance.md` | keeping this repository accurate |

## Skills

Skills are procedures. They are narrower than agents and usually apply to a
single task.

| Skill | Procedure |
|---|---|
| `skills/build-test.md` | run the correct build and test commands per repository |
| `skills/contract-change.md` | change a contract and propagate it safely |
| `skills/widget-authoring.md` | create, register and test a widget crate |
| `skills/layout-authoring.md` | write, validate and publish a layout |
| `skills/plugin-authoring.md` | create and wire a native plugin |
| `skills/runtime-widgets.md` | build and install WASM or external widgets |
| `skills/release-core.md` | verify and package a kernel release |
| `skills/web-docs-sync.md` | sync upstream docs and translations into the site |
| `skills/context-maintenance.md` | update and audit this context repository |

## Usage

1. Pick the agent matching the task.
2. Read the files listed in its "Required reading" section.
3. Follow the matching skill for the mechanical steps.
4. Use `../AGENTS.md` and `../conventions.md` as global rules.
5. Finish with the repository's local CI and, when behavior changed, a
   context update.

## Rules that apply to every agent

- Upstream source wins over this context.
- No secrets, tokens, personal data or machine-specific paths.
- No emojis.
- No speculative API surface.
- All crates stay at `0.1.0`.
- Rust repositories keep local CI as their gate; CI is being added
  gradually (currently the kernel release workflow and roadmap sync, plus
  the web deployment).
- Never claim a check passed without running it.
