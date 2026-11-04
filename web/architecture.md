# Web architecture

The site is a static Next.js export. It has no server runtime and no
database.

## Rendering

- `next.config.mjs` sets `output: "export"`, `trailingSlash: true`,
  unoptimized images and an optional asset prefix.
- `PAGES_BASE_PATH` supplies the repository sub-path for GitHub Pages.
- `NEXT_PUBLIC_ASSET_PREFIX` is used by client code to build asset and link
  URLs that work under the sub-path.

## App Router layout

```
app/
  (es)/
    layout.tsx
    page.tsx                 Spanish landing
    not-found.tsx
    docs/es/[[...slug]]/page.tsx
  (en)/
    layout.tsx
    page.tsx                 English landing
    en/page.tsx
    not-found.tsx
    docs/[[...slug]]/page.tsx
    docs/en/[[...slug]]/page.tsx
  globals.css
  fonts/
components/
  landing.tsx
  docs-shell.tsx
  doc-view.tsx
  docs-top-search.tsx
  site-header.tsx
  site-footer.tsx
  theme-toggle.tsx
  use-locale.ts
  page-shell.tsx
lib/
  docs.ts
  i18n.ts
  repos.ts
  search.tsx
  search-data.ts
  asset.ts
  palettes.ts
```

The Spanish landing is the default route. English lives under `/en/`.
Documentation is nested under `/docs/<locale>/`.

## Documentation pipeline

`scripts/sync-docs.sh` copies from sibling checkouts:

- `README.md` from each repository;
- the `docs/` tree;
- selected extra files:
  - `custom/README.md` for widgets;
  - `layouts/custom/README.md` for layouts;
  - the samurai README and changelog for plugins;
  - the MCP README for extensions;
  - `ROADMAP.md`, `CHANGELOG.md` and `CONTRIBUTING.md` for the kernel.

At build time, `lib/docs.ts`:

1. walks the selected locale root;
2. builds an index of Markdown files per repository;
3. ranks files using a curated order per repository;
4. derives a title from the first H1, falling back to a generated label;
5. maps each file to a URL slug;
6. renders Markdown to HTML with `marked`;
7. rewrites relative links to site documentation routes or GitHub blob
   URLs;
8. rewrites preview image references to local WebP assets;
9. wraps tables for horizontal scrolling.

`lib/repos.ts` is the single description of the seven documented
repositories: key, display name, role, kind, localized description and
feature tags.

## Search

- `scripts/build-search-index.mjs` runs before the build.
- It produces `lib/search-data.ts` with text, title, path, label, repo and
  locale for every document.
- `lib/search.tsx` ranks documents by weighted term occurrences: title
  matches dominate, then path, then label, then body frequency.
- Queries shorter than two characters return no results.
- Results are capped at 12.
- Snippets are word-clipped windows around the first match and matches are
  highlighted with a `mark` element.

## Internationalization

`lib/i18n.ts` defines the locale model and all landing copy.

- `DEFAULT_LOCALE` is Spanish.
- `/docs/es/*` is Spanish.
- `/en*`, `/docs/en/*` and `/docs/*` are English.
- The function `stripSubPath` removes the deployment prefix before locale
  detection.
- The landing copy is stored as typed objects for both locales.

## Assets

- `public/img/logo.*` and `public/img/previews/*.webp` are site assets.
- `lib/asset.ts` and `lib/i18n.ts` provide prefix-aware URL helpers.
- `next.config.mjs` disables image optimization for static export.

## Styling

- `app/globals.css` holds the site styles.
- The site uses the X palette in dark mode and the Madrid palette in light
  mode, sourced from `xscriptor-colors/assets`.
- Fonts are Hack Nerd Font Mono, self-hosted under `app/fonts/`.

## Constraints

- Static export only; no API routes, middleware or dynamic server code.
- All documentation must exist as Markdown at build time.
- Every internal link is resolved by `lib/docs.ts`; missing targets fall back
  to GitHub blob URLs.
