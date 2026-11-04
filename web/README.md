# web - site and documentation

The `web` repository is the landing page and documentation site for xtop.
It mirrors the documentation of the other seven repositories and publishes a
static site.

## Stack

- Next.js 15 App Router;
- React 19;
- TypeScript 5.8;
- `marked` for Markdown rendering;
- static export (`output: "export"`);
- GitHub Pages deployment.

## Routes

- `/` - Spanish landing page (default locale);
- `/en/` - English landing page;
- `/docs/es/...` - Spanish documentation;
- `/docs/en/...` - English documentation;
- `/docs/...` - legacy alias rendered as English.

## Documentation sources

- `docs/` - English mirror, copied from the sibling repositories;
- `i18n/es/docs/` - Spanish translation;
- `docs/_home.md` - authored landing content for the docs hub;
- `public/img/` - logo and previews;
- `scripts/build-search-index.mjs` - prebuild search index;
- `scripts/sync-docs.sh` - copies upstream Markdown into `docs/`.

## Commands

```
npm ci
npm run dev
npm run build
npm run start
```

`prebuild` regenerates the search index automatically before `build`.

## Documents

- `architecture.md` - rendering, docs pipeline, search, i18n, assets.
- `content.md` - docs mirror, translations, landing copy.
- `operations.md` - build, sync, translation and deployment procedures.

## Automation

`.github/workflows/deploy.yml` builds on push to `main` and deploys the
static output to GitHub Pages.
