# Web operations

## Local development

```
cd ~/repos/xtop-cli/web
npm ci
npm run dev
```

## Build

```
npm run prebuild
npm run build
```

`npm run build` runs `prebuild` automatically first.

- `prebuild` regenerates the search index.
- `build` produces the static export in `out/`.

## Sync upstream documentation

The script expects sibling checkouts under the parent directory:

```
cd ~/repos/xtop-cli/web
./scripts/sync-docs.sh
```

It copies:

- `README.md` and `docs/` from `xtop`, `api`, `effects`, `extensions`,
  `layouts`, `plugins` and `widgets`;
- the extra files listed in the script.

After syncing, the Spanish mirror must be updated by hand.

## Translation workflow

1. Sync the English mirror.
2. Compare `docs/` with `i18n/es/docs/` to find added, removed or renamed
   files.
3. Translate changed content.
4. Verify links and image references.
5. Rebuild and spot-check the Spanish docs routes.

## Deploy

`.github/workflows/deploy.yml`:

- triggers on push to `main` and manual dispatch;
- installs Node 22 with npm cache;
- runs `npm ci`;
- configures Pages;
- caches `.next/cache`;
- builds with `PAGES_BASE_PATH` and `NEXT_PUBLIC_ASSET_PREFIX` from the
  Pages configuration;
- uploads `out/` and deploys with the Pages action.

Permissions: `contents: read`, `pages: write`, `id-token: write`.
Concurrency group `pages`, no cancellation in progress.

## Deployment checklist

1. `./scripts/sync-docs.sh` after upstream docs changes.
2. Update the Spanish mirror.
3. `npm ci && npm run build`.
4. Verify landing pages in both languages.
5. Verify the docs hub, at least one nested doc per repository, and search.
6. Verify asset and preview images load under the base path.
7. Push to `main` and confirm the Pages workflow succeeds.

## Troubleshooting

- Blank docs page: the file is not in `docs/<repo>/` or not listed in
  `lib/repos.ts`; `resolveDoc` returns null.
- Broken internal link: the target file is missing from the mirror or the
  slug does not match; `lib/docs.ts` falls back to a GitHub blob URL.
- Search returns nothing: the prebuild index was not regenerated, or the
  query is shorter than two characters.
- Images broken on the deployed site: the asset prefix is missing; use the
  `asset()` helper.
- Locale wrong under a sub-path: `stripSubPath` expects the configured
  prefix.

## Constraints

- Do not add server-side code; the output must remain a static export.
- Do not edit `docs/` as the source of truth for upstream material.
- Do not commit `node_modules/`, `.next/` or `out/`.
- Do not store tokens or deployment credentials in the repository; the
  workflow uses the repository token and Pages OIDC.
