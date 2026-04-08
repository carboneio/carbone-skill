# Contributing

## What belongs in this repository

This repository contains the official Carbone skill — a set of Markdown files that teach AI assistants the Carbone templating syntax. Contributions should improve the accuracy, completeness, or clarity of that documentation.

Good contributions:
- Correcting a documented syntax that is wrong or outdated
- Adding a missing formatter, option, or behaviour that is already shipped in Carbone
- Improving an example to make it clearer or more representative
- Fixing a typo or broken formatting

Out of scope:
- Feature requests for Carbone itself — open those at [carbone.io](https://carbone.io)
- Adding undocumented or unreleased behaviour

## How to contribute

1. Fork the repository and create a branch from `main`
2. Make your changes in the relevant file:
   - `carbone/SKILL.md` — core syntax (tags, loops, conditions, `:set`, i18n, options)
   - `carbone/references/formatters.md` — formatter reference
   - `carbone/references/html-templates.md` — HTML template reference
   - `carbone/references/markdown-templates.md` — Markdown template reference
3. Open a pull request with a short description of what changed and why

## Reporting issues

Use the issue templates:
- **Documentation error** — for syntax that is documented incorrectly or is outdated
- **Missing coverage** — for a formatter or feature that exists in Carbone but is not documented in the skill

Before opening an issue, check the [Carbone changelog](https://carbone.io/changelog.html) to confirm the feature is in a released version.

## License

By contributing, you agree that your contributions will be licensed under the [Apache-2.0 license](LICENSE).
