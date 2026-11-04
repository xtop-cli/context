# Skill: web docs sync

## 1. Check upstream state

```
for repo in xtop api effects extensions layouts plugins widgets; do
  echo "== $repo"
  git -C ~/repos/xtop-cli/$repo log -1 --oneline
done
```

## 2. Sync the English mirror

```
cd ~/repos/xtop-cli/web
./scripts/sync-docs.sh
```

The script copies `README.md`, `docs/` and the extra files declared in its
`EXTRA` table.

## 3. Review the diff

```
git -C ~/repos/xtop-cli/web status --short docs
git -C ~/repos/xtop-cli/web diff --stat -- docs
```

Look for:

- new and removed files;
- renamed documents;
- changed H1 titles;
- new image references;
- links to files that do not exist in the mirror.

## 4. Update the Spanish mirror

Compare trees:

```
diff -rq ~/repos/xtop-cli/web/docs ~/repos/xtop-cli/web/i18n/es/docs
```

For each changed file:

1. create or rename the Spanish counterpart;
2. translate the content;
3. preserve code blocks, tables and links;
4. verify relative links resolve inside the Spanish tree.

## 5. Update the index if needed

Adding a repository or a curated document requires edits in:

- `lib/repos.ts`;
- `CURATED` in `lib/docs.ts`;
- `REPOS` and `EXTRA` in `scripts/sync-docs.sh`.

## 6. Build

```
cd ~/repos/xtop-cli/web
npm ci
npm run build
```

## 7. Verify

- `/` and `/en/` render;
- docs hub lists every repository;
- one nested document per repository renders;
- search returns hits for a known term;
- previews and logo load under the base path;
- no broken heading anchors in the changed documents.

## 8. Deploy

Push to `main`. The Pages workflow builds and deploys.

## 9. Report

List synced repositories, translated files, routes checked and any document
that remains untranslated or stale.
