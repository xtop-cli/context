# Web content model

## Repositories presented

`lib/repos.ts` lists seven repositories. The context repository itself is
not part of the public site.

| Key | Display | Role | Kind |
|---|---|---|---|
| `xtop` | Xtop | kernel | App |
| `api` | api | contracts | Workspace |
| `widgets` | widgets | renderers | Workspace |
| `layouts` | layouts | arrangement | Crate |
| `plugins` | plugins | functionality | Workspace |
| `extensions` | extensions | kernel hooks | Workspace |
| `effects` | effects | animation | Workspace |

`web` is the site itself and is not listed as a documented repository.

## Curated document order

`lib/docs.ts` defines the display order per repository.

Kernel:

1. `README.md`
2. `docs/features.md`
3. `docs/installation.md`
4. `docs/usage.md`
5. `docs/configuration.md`
6. `docs/customization.md`
7. `docs/colors.md`
8. `docs/plugin.md`
9. `docs/multi-repo.md`
10. `docs/design.md`
11. `ROADMAP.md`
12. `CHANGELOG.md`
13. `CONTRIBUTING.md`

API:

1. `README.md`
2. `docs/architecture.md`
3. `docs/data-model.md`
4. `docs/plugin-contract.md`
5. `docs/widget-contract.md`
6. `docs/extension-contract.md`
7. `docs/effect-contract.md`
8. `docs/changes.md`

Widgets:

1. `README.md`
2. `docs/widgets.md`
3. `docs/authoring.md`
4. `custom/README.md`

Layouts:

1. `README.md`
2. `docs/layout-schema.md`
3. `docs/authoring.md`
4. `docs/decisions.md`
5. `custom/README.md`

Plugins:

1. `README.md`
2. `xtop-plugin-samurai/README.md`
3. `docs/architecture.md`
4. `docs/rules.md`

Extensions:

1. `README.md`
2. `xtop-extension-mcp/README.md`
3. `docs/architecture.md`
4. `docs/mcp-protocol.md`

Effects:

1. `README.md`
2. `docs/effects.md`
3. `docs/write-effect.md`

Files outside the curated list are still indexed and rendered, ordered by a
grade: README first, then `docs/`, then extras.

## English mirror

`docs/` is the canonical English mirror. It is a copy, not a second source:
upstream files are authoritative. Update it with
`scripts/sync-docs.sh`, never by editing the mirror as if it were the
source.

## Spanish mirror

`i18n/es/docs/` mirrors the English tree structure. Translations are
maintained by hand. When a file is added or renamed upstream:

1. run the sync script;
2. add or rename the corresponding Spanish file;
3. translate the content;
4. verify internal links.

The Spanish source is not generated.

## Docs hub

`docs/_home.md` is authored in the web repository and is not synced. It is
the documentation landing content.

## Landing copy

`lib/i18n.ts` holds all marketing copy for both locales:

- navigation and theme toggles;
- hero title and lede;
- badges and statistics;
- feature grid;
- screenshot captions;
- palette section;
- ecosystem section;
- install panels and notes;
- essential key list;
- call to action and footer.

The statistics mention seven repositories, a documentation file count,
twelve themes, eleven base widgets, twelve samurai actions, ten heuristic
rules and a 500 ms fade.

When a count or a feature changes upstream, update the copy in both
locales.

## Adding a repository to the site

1. Add an entry to `REPOS` in `lib/repos.ts`.
2. Add a curated order for it in `CURATED` in `lib/docs.ts`.
3. Add the repository to `REPOS` in `scripts/sync-docs.sh` and any extra
   paths.
4. Mirror its docs into `docs/<key>/`.
5. Add the Spanish mirror under `i18n/es/docs/<key>/`.
6. Rebuild and verify navigation and search.
7. Update this context.

A repository that is not listed in `lib/repos.ts` is not rendered even if
its files exist under `docs/`.
