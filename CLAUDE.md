# Carbone Skill — Developer Notes

## Before publishing a new version

- [ ] Are the "Read this file when…" trigger lines on every reference file accurate and up to date?
- [ ] Is the skill optimised for AI agents? (clear triggers, no ambiguous language, anti-hallucination guard in place)
- [ ] Does the skill follow the latest Claude Code skill specification? (frontmatter fields, line count, description+when_to_use char budget)
- [ ] Is `CHANGELOG.md` updated with all changes under the correct version?
- [ ] Is the version number bumped consistently across all files?
  - `carbone/SKILL.md` — `metadata.version`
  - `.claude-plugin/plugin.json` — `version`
  - `.claude-plugin/marketplace.json` — `metadata.version` and `plugins[0].version`
