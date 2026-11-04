# Skill: context maintenance

## 1. Establish the upstream delta

```
for repo in api effects extensions layouts plugins web widgets xtop; do
  echo "== $repo"
  git -C ~/repos/xtop-cli/$repo log -1 --oneline
  git -C ~/repos/xtop-cli/$repo status --short
done
```

Read the changed files, not only the commit messages.

## 2. Map the change to documents

| Upstream change | Context file |
|---|---|
| Workspace member added or removed | `repositories.md`, repo folder |
| New command or flag | `xtop/cli.md` |
| Config key | `xtop/configuration.md` |
| Theme format or count | `xtop/theming.md` |
| Layout schema or default | `layouts/schema.md`, `layouts/defaults.md` |
| Widget name or option | `widgets/widget-reference.md` |
| Contract type or behavior | `api/` matching document |
| Plugin action or rule | `plugins/samurai.md`, `plugins/rules.md` |
| Runtime widget protocol | `plugins/runtime-widgets.md` and the host doc |
| MCP tool or method | `extensions/mcp-protocol.md` |
| Effect semantics | `effects/catalog.md` |
| Web route or pipeline | `web/architecture.md` |
| Distribution change | `xtop/operations.md`, `workflows.md` |
| Status or deferred work | `current-state.md` |
| New or renamed document | `llms.txt`, section indexes |

## 3. Edit narrowly

- Update the smallest set of files.
- Summarize; do not paste whole upstream documents.
- Keep the upstream document reference.
- Use the `~/repos/xtop-cli/<repo>` path convention.
- No emojis, secrets, personal data or machine-specific paths.

## 4. Run consistency checks

```
grep -rn "0\.1\.0\|1\.87\|0\.30\.2" .
grep -rn "xtop-cli/" .
grep -rn "TODO\|FIXME\|TBD" .
```

Check:

- theme count is twelve;
- base widget count is eleven;
- samurai actions are twelve;
- heuristic rules are ten;
- detail presets are three;
- default layouts are ten;
- repositories are eight plus this context.

## 5. Verify links

Every relative Markdown link must resolve to a file in this repository. Use a
simple link checker or a manual pass over the files you edited.

## 6. Update the index

If you added, removed or renamed a file:

- update `README.md` document map;
- update `llms.txt`;
- update the relevant section index;
- update `resources/README.md` if an agent or skill changed.

## 7. Review

- Does any claim contradict the code?
- Does any document describe deferred work as implemented?
- Does any example use a path or name that does not exist?
- Is anything sensitive present?

## 8. Report

State the upstream change, the files updated, the checks run and any
remaining uncertainty.
