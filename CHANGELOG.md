# Changelog

## 1.3.2
- New patterns: extended `:and`/`:or` argument forms (`.prop`, absolute `d.`/`c.`, `$alias.field`); alias array aggregation (`{$alias[].field:aggSum}`); alias arrays inside native DOCX chart cells
- `SKILL.md` item 22: acknowledged filter-expression alias exception (bare field name allowed when alias body is a filter expression)
- `:transform`: added `in` unit (v5.4.0+ for PPTX/ODP)
- `:html` compatibility: added HTML to the format list
- `:drop`/`:keep` elements: unified element list and per-format support across `SKILL.md`, `formatters.md`, `html-templates.md`; clarified the `N` argument applies to `p` and `row` only
- `:ellipsis`: clarified `maximum` is the truncation point, not the total output length
- UNRECOMMENDED number formatters now ship inline replacement guidance: `:int` → `:add(0)` / `:abs`; `:toFixed` → `:round`; `:toEN`/`:toFR` → `:formatN` with `lang`
- ECharts: recommended `echarts@v5a` as default, `echarts@v5` as legacy
- Removed deprecated `:convDate` from the main Date Formatters table
- Audit fixes: corrected invalid `d[i=0]invoice[i=0]` alias-shorthand claim; corrected `:barcode` options syntax (comma-separated `key:value`, not semicolon-separated string); switched address-block country example to `:ifEQ` for correct equality semantics; removed stray spaces around `=` in array-filter example; fixed two broken markdown table separators in `formatters.md`

## 1.3.1
- New patterns in reference files: `:t:aggStrD`, post-aggregation arithmetic, datetime normalization in-place (`:startOfD:formatD:set` and `:substr:set`), chart from indexed array with `:imageFit`
- Added `{o.preReleaseFeatureIn=4025011}` to `references/runtime-options.md`
- `SKILL.md`: universal formatter argument quoting rule (item 6), loop iterator casing best practice (§9), `user-invocable: true`
- `extract-carbone-tags.py`: added `:image(width=N,height=N)` to `INVALID_FORMATTERS`

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