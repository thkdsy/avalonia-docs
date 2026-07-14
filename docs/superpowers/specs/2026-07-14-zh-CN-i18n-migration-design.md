# zh-CN i18n Migration Design

## Goal

Add Chinese (zh-CN) language support to the Avalonia docs site using Docusaurus i18n, migrating existing Chinese translations from an older fork.

## Source

- **Old Chinese docs:** `E:\Desktop\avalonia示例\avalonia-docs-Cn\` (direct in-place translation of docs, no i18n)
- **New English docs:** current working directory (upstream English source)

## Architecture

### i18n Configuration

Add to `docusaurus.config.ts`:

```ts
i18n: {
  defaultLocale: 'en',
  locales: ['en', 'zh-CN'],
  localeConfigs: {
    en: { label: 'English' },
    'zh-CN': { label: '中文' },
  },
},
```

Add locale dropdown to navbar items.

### Directory Structure

Chinese translations go under `i18n/zh-CN/`, mirroring each doc plugin:

```
i18n/zh-CN/
├── docusaurus-plugin-content-docs/
│   └── current/                    ← docs/ (versioned, "current" = latest)
├── docusaurus-plugin-content-docs-controls/
│   └── current/                    ← controls/
├── docusaurus-plugin-content-docs-tools/
│   └── current/                    ← tools/
├── docusaurus-plugin-content-docs-troubleshooting/
│   └── current/                    ← troubleshooting/
├── docusaurus-plugin-content-docs-xpf/
│   └── current/                    ← xpf/
└── docusaurus-theme-classic/       ← navbar/footer UI strings
```

API docs (`api/` and `api_versioned_docs/`) are **not translated** — they are auto-generated reference pages.

### Scope

| Directory | Files (old) | Files (new) | Overlap | New to translate |
|-----------|------------|------------|---------|-----------------|
| docs | 233 | 233 | 231 | 2 |
| controls | 102 | 232 | 92 | 140 |
| tools | 32 | 33 | 31 | 2 |
| troubleshooting | 11 | 14 | 11 | 3 |
| xpf | 29 | 30 | 29 | 1 |
| api + api_versioned_docs | N/A | N/A | N/A | Skip (auto-generated) |

**Total new files to translate: ~148** (mostly new chart controls documentation)

## Migration Strategy

### Phase 1: Configuration

1. Add `i18n` block to `docusaurus.config.ts`
2. Add locale dropdown to navbar
3. Create navbar/footer translations in `i18n/zh-CN/docusaurus-theme-classic/`

### Phase 2: Content Migration

For each content directory (`docs`, `controls`, `tools`, `troubleshooting`, `xpf`):

1. Iterate over all markdown files in the old Chinese directory
2. If a file with the same relative path exists in the new English directory → copy Chinese version to the corresponding i18n path
3. If a file exists in old but NOT in new → copy to i18n anyway (may have been renamed/removed upstream, preserve the translation as it may still be useful)
4. Files in new but NOT in old → translate to Chinese using AI, then place in i18n

### Phase 3: Translation of New Content

For files present in the new English version but absent from the old Chinese version:

1. Identify the delta per directory
2. Translate each file using AI, matching the tone and terminology of the existing Chinese translations
3. Place translated files in the corresponding i18n path

### Phase 4: Verification

1. Run `npm run build` to verify the site builds with both locales
2. Spot-check pages in both English and Chinese
3. Verify language switcher works

## Notes

- **controls/ has the largest delta**: 140 new files, mostly chart controls documentation added upstream after the old Chinese fork. These will be translated via AI.
- **API docs are skipped**: `api/` and `api_versioned_docs/` contain auto-generated reference pages. Translating them is impractical and they change with every release.
- **Upstream config conflicts**: Only `docusaurus.config.ts` may conflict on pull. The i18n block is small and self-contained — easy to resolve manually.

## Upstream Update Strategy

When pulling from upstream (`AvaloniaUI/avalonia-docs`):

1. `git pull upstream main` — English docs update normally
2. Chinese translations in `i18n/zh-CN/` are untouched
3. `docusaurus.config.ts` may have minor conflicts in the i18n block (manually resolve)
4. Check diff for new English files → translate and add to i18n
5. Check diff for changed English files → manually update corresponding Chinese translations if needed

## Affected Files

### Modified
- `docusaurus.config.ts` — add i18n config + locale dropdown

### Created
- `i18n/zh-CN/` — entire directory tree with Chinese translations
- Migration script (one-off, deleted after use)

### Not Modified
- All English content in `docs/`, `controls/`, `tools/`, `troubleshooting/`, `xpf/`
- Sidebars, plugins, static assets, theme
