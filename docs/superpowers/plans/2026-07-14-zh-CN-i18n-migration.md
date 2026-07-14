# zh-CN i18n Migration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add Chinese (zh-CN) language support to the Avalonia docs site via Docusaurus i18n, migrating existing translations and translating new content.

**Architecture:** Docusaurus i18n with `en` as default locale and `zh-CN` as additional locale. Chinese translations live in `i18n/zh-CN/` under plugin-specific subdirectories. English content stays untouched.

**Tech Stack:** Docusaurus (React), TypeScript, Markdown/MDX

---

### Task 1: Add i18n configuration to docusaurus.config.ts

**Files:**
- Modify: `docusaurus.config.ts`

- [ ] **Step 1: Add i18n block after `projectName`**

In `docusaurus.config.ts`, find the line `projectName: 'avalonia',` (line 22). Insert the i18n config right after it:

```ts
  projectName: 'avalonia',
  i18n: {
    defaultLocale: 'en',
    locales: ['en', 'zh-CN'],
    localeConfigs: {
      en: { label: 'English' },
      'zh-CN': { label: '中文' },
    },
  },
  onBrokenLinks: 'warn',
```

- [ ] **Step 2: Add locale dropdown to navbar**

In `docusaurus.config.ts`, add a locale dropdown item to the navbar items array. Insert it before the search item in the navbar:

```ts
        {
          type: 'localeDropdown',
          position: 'right',
        },
```

Insert this right before:
```ts
        {
          type: 'search',
          position: 'right',
        },
```

- [ ] **Step 3: Commit**

```bash
git add docusaurus.config.ts
git commit -m "feat: add i18n config for zh-CN locale

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>"
```

---

### Task 2: Create i18n directory structure and theme translations

**Files:**
- Create: `i18n/zh-CN/docusaurus-theme-classic/navbar.json`
- Create: `i18n/zh-CN/code.json`

- [ ] **Step 1: Create directory structure**

```bash
mkdir -p i18n/zh-CN/docusaurus-theme-classic
mkdir -p i18n/zh-CN/docusaurus-plugin-content-docs/current
mkdir -p i18n/zh-CN/docusaurus-plugin-content-docs-controls/current
mkdir -p i18n/zh-CN/docusaurus-plugin-content-docs-tools/current
mkdir -p i18n/zh-CN/docusaurus-plugin-content-docs-troubleshooting/current
mkdir -p i18n/zh-CN/docusaurus-plugin-content-docs-xpf/current
```

- [ ] **Step 2: Create navbar translations**

Write `i18n/zh-CN/docusaurus-theme-classic/navbar.json`:

```json
{
  "title": {
    "message": "Avalonia 文档",
    "description": "Navbar title"
  },
  "logo.alt": {
    "message": "Avalonia Logo",
    "description": "Navbar logo alt text"
  },
  "item.label.Guides": {
    "message": "指南",
    "description": "Navbar item Guides"
  },
  "item.label.Controls": {
    "message": "控件",
    "description": "Navbar item Controls"
  },
  "item.label.Tools": {
    "message": "工具",
    "description": "Navbar item Tools"
  },
  "item.label.APIs": {
    "message": "API",
    "description": "Navbar item APIs"
  },
  "item.label.More": {
    "message": "更多",
    "description": "Navbar item More"
  },
  "item.label.Troubleshooting": {
    "message": "故障排查",
    "description": "Navbar item Troubleshooting"
  },
  "item.label.Community Translations": {
    "message": "社区翻译",
    "description": "Navbar item Community Translations"
  },
  "item.label.Enhanced Support": {
    "message": "增强支持",
    "description": "Navbar item Enhanced Support"
  },
  "item.label.Professional Services": {
    "message": "专业服务",
    "description": "Navbar item Professional Services"
  },
  "item.label.GitHub Discussions": {
    "message": "GitHub 讨论",
    "description": "Navbar item GitHub Discussions"
  },
  "item.label.Blog": {
    "message": "博客",
    "description": "Navbar item Blog"
  },
  "item.label.Avalonia XPF": {
    "message": "Avalonia XPF",
    "description": "Navbar item Avalonia XPF"
  }
}
```

- [ ] **Step 3: Create code.json for the "Edit this page" label**

Write `i18n/zh-CN/code.json`:

```json
{
  "theme.common.editThisPage": {
    "message": "在 GitHub 上编辑此页",
    "description": "Edit this page link text"
  },
  "theme.common.skipToMainContent": {
    "message": "跳转到主要内容",
    "description": "Skip to main content"
  },
  "theme.SearchBar.label": {
    "message": "搜索",
    "description": "Search bar label"
  },
  "theme.SearchBar.seeAll": {
    "message": "查看全部 {count} 条结果",
    "description": "See all search results"
  },
  "theme.SearchPage.documentsFound.plurals": {
    "message": "找到 {count} 份文档",
    "description": "Search results count"
  },
  "theme.SearchPage.noResultsText": {
    "message": "未找到相关文档",
    "description": "No search results"
  },
  "theme.LastUpdated.atDateByAuthor": {
    "message": "最后更新于 {date}，作者 {author}",
    "description": "Last updated at date by author"
  },
  "theme.NotFound.title": {
    "message": "页面未找到",
    "description": "404 page title"
  },
  "theme.NotFound.p1": {
    "message": "你访问的页面不存在。",
    "description": "404 page description"
  },
  "theme.admonition.note": {
    "message": "备注",
    "description": "Admonition note title"
  },
  "theme.admonition.tip": {
    "message": "提示",
    "description": "Admonition tip title"
  },
  "theme.admonition.info": {
    "message": "信息",
    "description": "Admonition info title"
  },
  "theme.admonition.warning": {
    "message": "警告",
    "description": "Admonition warning title"
  },
  "theme.admonition.danger": {
    "message": "危险",
    "description": "Admonition danger title"
  },
  "theme.blog.paginator.navAriaLabel": {
    "message": "博客列表分页导航",
    "description": "Blog paginator label"
  },
  "theme.blog.paginator.newerEntries": {
    "message": "较新的文章",
    "description": "Newer blog entries"
  },
  "theme.blog.paginator.olderEntries": {
    "message": "较旧的文章",
    "description": "Older blog entries"
  },
  "theme.docs.paginator.navAriaLabel": {
    "message": "文档分页导航",
    "description": "Docs paginator label"
  },
  "theme.docs.paginator.previous": {
    "message": "上一页",
    "description": "Previous doc page"
  },
  "theme.docs.paginator.next": {
    "message": "下一页",
    "description": "Next doc page"
  },
  "theme.docs.breadcrumbs.navAriaLabel": {
    "message": "面包屑导航",
    "description": "Breadcrumbs navigation label"
  },
  "theme.CodeBlock.copyButtonAriaLabel": {
    "message": "复制代码",
    "description": "Copy code button"
  },
  "theme.CodeBlock.copied": {
    "message": "已复制",
    "description": "Copied notification"
  },
  "theme.CodeBlock.wordWrapToggle": {
    "message": "切换自动换行",
    "description": "Word wrap toggle"
  },
  "theme.tags.tagsPageLink": {
    "message": "所有标签",
    "description": "Tags page link"
  },
  "theme.tags.tagsListLabel": {
    "message": "标签：",
    "description": "Tags list label"
  }
}
```

- [ ] **Step 4: Commit**

```bash
git add i18n/
git commit -m "feat: add zh-CN i18n directory structure and theme translations

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>"
```

---

### Task 3: Write and run migration script for existing translations

**Files:**
- Create: `scripts/migrate-zh-cn.sh` (one-off, delete after use)

- [ ] **Step 1: Write the migration script**

Write `scripts/migrate-zh-cn.sh`:

```bash
#!/bin/bash
set -e

OLD="E:/Desktop/avalonia示例/avalonia-docs-Cn"
I18N="i18n/zh-CN"

# Define source→target mappings: old_dir → i18n_subdir
declare -A MAPPINGS=(
  ["docs"]="$I18N/docusaurus-plugin-content-docs/current"
  ["controls"]="$I18N/docusaurus-plugin-content-docs-controls/current"
  ["tools"]="$I18N/docusaurus-plugin-content-docs-tools/current"
  ["troubleshooting"]="$I18N/docusaurus-plugin-content-docs-troubleshooting/current"
  ["xpf"]="$I18N/docusaurus-plugin-content-docs-xpf/current"
)

migrated=0
skipped=0

for dir in "${!MAPPINGS[@]}"; do
  target="${MAPPINGS[$dir]}"
  mkdir -p "$target"
  while IFS= read -r -d '' file; do
    rel="${file#$OLD/$dir/}"
    new_english="$(pwd)/$dir/$rel"
    if [ -f "$new_english" ]; then
      dest="$target/$rel"
      mkdir -p "$(dirname "$dest")"
      cp "$file" "$dest"
      echo "MIGRATED: $rel"
      ((migrated++)) || true
    else
      echo "SKIPPED (removed upstream): $rel"
      ((skipped++)) || true
    fi
  done < <(find "$OLD/$dir" -type f \( -name "*.md" -o -name "*.mdx" \) -print0)
done

echo ""
echo "Done. Migrated: $migrated, Skipped: $skipped"
```

- [ ] **Step 2: Run the migration script**

```bash
bash scripts/migrate-zh-cn.sh
```

Expected output: list of migrated files, with a few "SKIPPED" for files removed upstream. Confirm migrated count matches expectations (~394 files across 5 directories).

- [ ] **Step 3: Delete the migration script**

```bash
rm scripts/migrate-zh-cn.sh
```

- [ ] **Step 4: Commit**

```bash
git add i18n/
git commit -m "feat: migrate existing zh-CN translations into i18n structure

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>"
```

---

### Task 4: Translate new docs/ files

**Files:**
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs/current/community-translations.mdx`

**Note:** The only new translated docs file is `community-translations.mdx`. Skip `docs/superpowers/specs/` — it's a project management file, not user-facing docs.

- [ ] **Step 1: Read the English source and translate**

Read `docs/community-translations.mdx`, then write the Chinese translation to `i18n/zh-CN/docusaurus-plugin-content-docs/current/community-translations.mdx`.

Translate the frontmatter (`title`, `description`) and all content. Keep code/link references unchanged. Maintain the same MDX syntax and structure.

- [ ] **Step 2: Commit**

```bash
git add i18n/zh-CN/docusaurus-plugin-content-docs/current/community-translations.mdx
git commit -m "feat: translate community-translations.mdx to zh-CN

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>"
```

---

### Task 5: Translate new controls/ files — charts batch 1 (analytics + bubble)

**Files:**
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/analytics/*.md` (14 files)
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/bubble/*.md` (3 files)

Files to translate:
- `controls/data-display/charts/analytics/bullet-chart.md`
- `controls/data-display/charts/analytics/bump-chart.md`
- `controls/data-display/charts/analytics/calendar-heatmap-chart.md`
- `controls/data-display/charts/analytics/funnel-chart.md`
- `controls/data-display/charts/analytics/heatmap-chart.md`
- `controls/data-display/charts/analytics/kpi-card.md`
- `controls/data-display/charts/analytics/matrix-chart.md`
- `controls/data-display/charts/analytics/pictorial-bar-chart.md`
- `controls/data-display/charts/analytics/pyramid-chart.md`
- `controls/data-display/charts/analytics/slope-chart.md`
- `controls/data-display/charts/analytics/table-chart.md`
- `controls/data-display/charts/analytics/theme-river-chart.md`
- `controls/data-display/charts/analytics/waffle-chart.md`
- `controls/data-display/charts/analytics/word-cloud-chart.md`
- `controls/data-display/charts/bubble/bubble-chart.md`
- `controls/data-display/charts/bubble/bubble-cloud-chart.md`
- `controls/data-display/charts/bubble/packed-bubble-chart.md`

- [ ] **Step 1: Translate all 17 files**

For each English source file in `controls/data-display/charts/analytics/` and `controls/data-display/charts/bubble/`:

1. Read the English source
2. Translate to Chinese, following the style seen in existing translations:
   - Translate `title` and `description` in frontmatter
   - Keep `id`, `doc-type`, `tags` unchanged
   - Translate body content, keeping code blocks unchanged
   - Translate section headings
3. Write to `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/analytics/<filename>` (same filename)

- [ ] **Step 2: Commit**

```bash
git add i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/analytics/
git add i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/bubble/
git commit -m "feat: translate chart docs (analytics + bubble) to zh-CN

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>"
```

---

### Task 6: Translate new controls/ files — charts batch 2 (cartesian + circular + comparison)

**Files:**
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/cartesian/*.md` (19 files)
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/circular/*.md` (3 files)
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/comparison/*.md` (8 files)

Total: 30 files. Translate all from `controls/data-display/charts/cartesian/`, `circular/`, `comparison/`.

- [ ] **Step 1: Translate all 30 files**

Same translation process as Task 5. For each English source file, read, translate, write to corresponding i18n path.

- [ ] **Step 2: Commit**

```bash
git add i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/cartesian/
git add i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/circular/
git add i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/comparison/
git commit -m "feat: translate chart docs (cartesian + circular + comparison) to zh-CN

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>"
```

---

### Task 7: Translate new controls/ files — charts batch 3 (engineering + financial + gauges)

**Files:**
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/engineering/*.md` (5 files)
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/financial/*.md` (9 files)
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/gauges/*.md` (6 files)

Total: 20 files.

- [ ] **Step 1: Translate all 20 files**

- [ ] **Step 2: Commit**

```bash
git add i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/engineering/
git add i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/financial/
git add i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/gauges/
git commit -m "feat: translate chart docs (engineering + financial + gauges) to zh-CN

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>"
```

---

### Task 8: Translate new controls/ files — charts batch 4 (hierarchy + index + maps + radial)

**Files:**
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/hierarchy/*.md` (18 files)
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/index.md` (1 file)
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/maps/*.md` (5 files)
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/radial/*.md` (6 files)

Total: 30 files.

- [ ] **Step 1: Translate all 30 files**

- [ ] **Step 2: Commit**

```bash
git add i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/hierarchy/
git add i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/index.md
git add i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/maps/
git add i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/radial/
git commit -m "feat: translate chart docs (hierarchy + maps + radial) to zh-CN

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>"
```

---

### Task 9: Translate new controls/ files — charts batch 5 (scheduling + shared-elements + statistical)

**Files:**
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/scheduling/*.md` (5 files)
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/shared-elements/*.md` (10 files)
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/statistical/*.md` (10 files)

Total: 25 files.

- [ ] **Step 1: Translate all 25 files**

- [ ] **Step 2: Commit**

```bash
git add i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/scheduling/
git add i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/shared-elements/
git add i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/charts/statistical/
git commit -m "feat: translate chart docs (scheduling + shared-elements + statistical) to zh-CN

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>"
```

---

### Task 10: Translate remaining new controls/ files (non-chart)

**Files:**
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/structured-data/tableview.md` (1 file)
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/text-display/markdown/codehighlighter.md` (1 file)
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/text-display/markdown/imageloader.md` (1 file)
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/text-display/markdown/index.md` (1 file)
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/text-display/markdown/markdown-styling.md` (1 file)
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/input/text-input/richtexteditor/*.md` (5 files)
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/input/text-input/virtualkeyboard/*.md` (3 files)
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/media/mediaplayer/*.md` (4 files)

Total: 17 files (140 - 123 charts translated in tasks 5-9).

- [ ] **Step 1: Translate all 17 non-chart files**

- [ ] **Step 2: Commit**

```bash
git add i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/structured-data/
git add i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/data-display/text-display/markdown/
git add i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/input/text-input/richtexteditor/
git add i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/input/text-input/virtualkeyboard/
git add i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/media/mediaplayer/
git commit -m "feat: translate remaining controls docs to zh-CN

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>"
```

---

### Task 11: Translate new tools/ files

**Files:**
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-tools/current/ai-tools/charts-mcp.md`
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-tools/current/assigning-seats.md`

- [ ] **Step 1: Translate both files**

Read each English source from `tools/`, translate, write to i18n path.

- [ ] **Step 2: Commit**

```bash
git add i18n/zh-CN/docusaurus-plugin-content-docs-tools/current/
git commit -m "feat: translate new tools docs to zh-CN

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>"
```

---

### Task 12: Translate new troubleshooting/ and xpf/ files

**Files:**
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-troubleshooting/current/controls/numericupdown.md`
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-troubleshooting/current/controls/richtexteditor.md`
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-troubleshooting/current/login-issues.md`
- Create: `i18n/zh-CN/docusaurus-plugin-content-docs-xpf/current/deployment/native-aot.md`

- [ ] **Step 1: Translate all 4 files**

- [ ] **Step 2: Commit**

```bash
git add i18n/zh-CN/docusaurus-plugin-content-docs-troubleshooting/current/
git add i18n/zh-CN/docusaurus-plugin-content-docs-xpf/current/
git commit -m "feat: translate new troubleshooting and xpf docs to zh-CN

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>"
```

---

### Task 13: Build and verify

- [ ] **Step 1: Install dependencies (if needed)**

```bash
npm install
```

- [ ] **Step 2: Build the site**

```bash
npm run build
```

Expected: build succeeds with both locales. Watch for broken link warnings related to zh-CN pages.

- [ ] **Step 3: Verify zh-CN locale files exist**

```bash
ls i18n/zh-CN/docusaurus-plugin-content-docs/current/ | head -20
ls i18n/zh-CN/docusaurus-plugin-content-docs-controls/current/ | head -20
```

- [ ] **Step 4: Verify build output includes zh-CN**

```bash
ls dist/zh-CN/ 2>/dev/null && echo "zh-CN build output found"
```

- [ ] **Step 5: Commit any final fixes**

```bash
git add -A
git commit -m "chore: final adjustments after build verification

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>"
```

---

### Final check: Verify git log

```bash
git log --oneline -15
```
