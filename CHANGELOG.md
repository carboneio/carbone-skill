# Changelog

## Unreleased
- Added `references/upgrade-guide.md` — Carbone upgrade guide covering v4→v5 migration and v5.0→v5.1.1 Studio Web Component breaking changes

## 1.2.2
- Added HH:mm pattern for `:formatI` with ISO 8601 durations in `references/formatters.md`

## 1.2.1
- Added `.claude-plugin/plugin.json` manifest required by the marketplace CI scanner
- Added `skills: "./carbone"` path in `plugin.json` for direct plugin installation
- Improved `marketplace.json`: added `$schema`, `category`, `homepage`, `repository`, `license`; removed `strict: false` (no longer needed now that `plugin.json` exists)

## 1.2.0
- Updated for Carbone v5.5.0
- Added `col` element to drop/keep (Section 6c) with updated format limits for XLSX, PPTX, HTML, ODS
- Added `div` and `span` drop/keep elements for HTML templates
- Added HTML entities support in `:html` formatter (`references/advanced-features.md`)
- Added `{o.hardRefresh=true}` runtime option (v5.4.4+)
- Added `carbone_version` field to metadata

## 1.1.1
- Improved description: imperative phrasing, added "even without Carbone by name" trigger
- Added HTML and Markdown to the document format list in the description trigger

## 1.1.0
- Split SKILL.md into reference files (`loops-advanced.md`, `set-patterns.md`, `advanced-features.md`, `xlsx-tips.md`, `runtime-options.md`) for progressive disclosure
- Fixed frontmatter: `description` block scalar `|` → `>`, removed `compatibility` field, added `version` to metadata

## 1.0.0
- Initial release