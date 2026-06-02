# Carbone Skill — Developer Notes

## Before publishing a new version

### Content quality
- [ ] Are the "Read this file when…" trigger lines on every reference file accurate and up to date?
- [ ] Is the skill optimised for AI agents? Concretely: no invented syntax (every formatter shown appears in `references/formatters.md`); anti-hallucination guard present in `SKILL.md`; no `{{` Mustache braces; no smart quotes (`'` / `'` / `"` / `"`); all examples conform to `SKILL.md` §10 Validation Checklist; trigger phrasing unambiguous
- [ ] Does the skill follow the latest Claude Code skill specification? (frontmatter fields, line count, description+when_to_use char budget)

### Cross-file consistency
- [ ] Same claim, same value across files — version requirements (v3/v4/v5), format-support lists (DOCX/XLSX/ODT/…), formatter element/scope tables (`:drop`/`:keep`, `:color`, `:html` compat), deprecation labels, pre-release flags (`{o.preReleaseFeatureIn=...}`) agree across `SKILL.md` and every reference file
- [ ] All internal cross-references resolve — every `see references/X.md §Y` / `[label](path)` points to a file and section that actually exists

### Cross-check against official Carbone docs
- [ ] Spot-check formatter signatures, argument order, unit lists, and version requirements against carbone.io documentation for anything new or modified in this release — drift between the skill and official docs is the most common source of incorrect AI suggestions

### Versioning
- [ ] Pre-release check — confirm no alpha/beta/RC tags already exist for the version you're about to bump to (`gh release list -R carboneio/carbone-skill` or the GitHub releases page)
- [ ] `CHANGELOG.md` updated with all changes under the correct version (semver bump: patch = doc fixes only, minor = new patterns or rules, major = breaking changes to existing tags/syntax)
- [ ] Version number bumped consistently across all four files:
  - `carbone/SKILL.md` — `metadata.version`
  - `.claude-plugin/plugin.json` — `version`
  - `.claude-plugin/marketplace.json` — `metadata.version` and `plugins[0].version`
