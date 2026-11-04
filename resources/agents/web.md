# Agent: web

## Scope

The `web` repository: Next.js static site, documentation mirror, Spanish
translations, search index and deployment.

## Required reading

- `../../web/README.md`
- `../../web/architecture.md`
- `../../web/content.md`
- `../../web/operations.md`
- `../../conventions.md`

## Workflow

1. If upstream docs changed, run `scripts/sync-docs.sh` first.
2. Update the Spanish mirror for changed files.
3. If a repository or document was added, update `lib/repos.ts` and
   `CURATED` in `lib/docs.ts`.
4. Rebuild; `prebuild` regenerates the search index.
5. Verify both landing pages, the docs hub, nested docs and search.
6. Confirm assets resolve under the configured base path.
7. Push only when the Pages workflow can succeed.

## Guardrails

- The English mirror is a copy; never treat it as the source of truth.
- Do not add server-side code; the output is a static export.
- Do not commit `node_modules/`, `.next/` or `out/`.
- Do not store deployment credentials.
- Do not leave the Spanish mirror silently stale.
- Do not edit generated search data by hand.

## Verification

```
cd ~/repos/xtop-cli/web
npm ci
npm run build
```

Spot-check routes:

- `/` and `/en/`;
- `/docs/es/` and `/docs/en/`;
- one nested document per repository;
- search for a term present in an upstream document.

## Output

Report the synced repositories, translated files, routes checked, build
result and any mirror that remains stale.
