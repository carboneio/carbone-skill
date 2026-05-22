# Changelog

## Unreleased
- Added "Read this file when…" trigger lines to `references/formatters.md`, `references/html-templates.md`, and `references/markdown-templates.md`
- Sharpened trigger descriptions in `references/practical-examples.md`, `references/runtime-options.md`, and `references/aliases.md`

## 1.3.0
- Added `references/upgrade-guide.md` — Carbone upgrade guide covering v4→v5 migration and v5.0→v5.1.1 Studio Web Component breaking changes
- Added `references/aliases.md` — production alias patterns: filter aliases, object-pick, frozen-index, loop shorthand, alias with fallback/arithmetic, looping over alias arrays
- Added `references/practical-examples.md` — practical examples: date/time formatting combinations, optional address blocks, checkbox patterns, `:ifEQ(NaN)` guard, range checks with `:and`, invoice aggregation chains, complement data `{c.}` patterns, `..` relative path inside formatter arguments
- Expanded Day.js token table in `references/formatters.md` to all 40 tokens organized by category; added weekday-conditional block pattern (`{c.now:formatD(d):ifEQ(N):showBegin/showEnd}`)
- Extended `references/practical-examples.md` with new patterns: `:add(0)` coercion before aggregation for string-encoded numbers; `abs():set()` / `div(-1):set()` for debit/credit display branching; `:aggMax` with a filter referencing a previously `:set` value via `..`; chained `:add` for summing sibling fields; `:prepend():append():html` order-of-operations
- Updated `SKILL.md` validation checklist: merged double-quote rules (item 18), added spaces-in-tags rule (item 23), added double-colon `::` invalidity rule (item 24)
- Updated `SKILL.md` frontmatter: added `when_to_use` trigger phrases, set `user-invocable: false` (background knowledge — Claude loads automatically, not a slash command)
- Added prominent anti-hallucination warning to `SKILL.md`: Carbone is not JSONPath, Mustache, Handlebars, or Jinja2 — only documented syntax is valid

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