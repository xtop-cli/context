# Agent: context maintenance

## Scope

This context repository. The agent keeps the derived documentation accurate
against upstream repositories.

## Required reading

- `../../README.md`
- `../../AGENTS.md`
- `../../conventions.md`
- `../../repositories.md`
- `../../current-state.md`
- `../skills/context-maintenance.md`

## Workflow

1. Identify the upstream change.
2. Read the upstream source and docs, not just the diff summary.
3. Update the narrowest affected file.
4. Update `repositories.md` if members, commands or automation changed.
5. Update `current-state.md` if status changed.
6. Update `llms.txt` if files were added, removed or renamed.
7. Run a consistency search for stale names, counts, versions and paths.
8. Verify internal links point at existing files.

## Guardrails

- Upstream source is authoritative.
- Do not copy whole upstream documents; summarize and link.
- Do not include secrets, tokens, personal data or machine paths.
- Do not include emojis.
- Do not present deferred work as implemented.
- Do not add a document without indexing it.

## Consistency checks

Search for:

- old crate or widget names;
- theme and layout counts;
- action and rule counts;
- `ratatui` and `rust-version` values;
- repository names and workspace members;
- absolute local paths outside the documented `~/repos/xtop-cli/` form;
- references to workflows or releases that do not exist.

## Output

Report the upstream change, the files updated, the checks run and any
remaining uncertainty.
