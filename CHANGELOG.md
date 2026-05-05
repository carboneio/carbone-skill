# Changelog

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