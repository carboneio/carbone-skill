# Carbone Skill — Official Carbone Templating Syntax for AI Assistants

Teaches AI assistants the Carbone Universal Templating Language — so they can design templates from scratch or embed Carbone tags into generated DOCX, XLSX, and PPTX files.

## Compatibility

Works with any platform that supports the open [Agent Skills](https://agentskills.io) standard:

| Platform | Instructions |
|---|---|
| Claude (claude.ai) | [docs.claude.com](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) |
| Claude Code | [code.claude.com](https://code.claude.com/docs/en/skills) |
| ChatGPT | [developers.openai.com](https://developers.openai.com/codex/skills/) |
| Cursor | [cursor.com/docs](https://cursor.com/docs/context/skills) |
| VS Code (GitHub Copilot) | [code.visualstudio.com](https://code.visualstudio.com/docs/copilot/customization/agent-skills) |
| Gemini CLI | [geminicli.com/docs](https://geminicli.com/docs/cli/skills/) |
| OpenAI Codex | [developers.openai.com](https://developers.openai.com/codex/skills/) |
| All other platforms | [agentskills.io](https://agentskills.io/home) |

## Download

**[carbone.skill](https://github.com/carboneio/carbone-skill/releases/latest/download/carbone.skill)** — latest release

## Installation

### Claude.ai

1. Go to **Customize → Skills** (at claude.ai/customize/skills)
2. Click **+** → **Upload a skill** and upload `carbone.skill`

### Claude Code

Register the repository as a plugin marketplace:

```
/plugin marketplace add carboneio/carbone-skill
```

Then install the skill:

```
/plugin install carbone@carbone-skill
```

Or install manually by extracting `carbone.skill`:

```bash
unzip carbone.skill -d ~/.claude/skills/carbone
```

### Cursor

See [cursor.com/docs/context/skills](https://cursor.com/docs/context/skills) for the exact install path.

### VS Code (GitHub Copilot)

```bash
unzip carbone.skill -d ~/.copilot/skills/carbone
```

### Gemini CLI

```bash
gemini skills install carbone.skill
```

### OpenAI Codex

```bash
unzip carbone.skill -d ~/.agents/skills/carbone
```

### Other platforms

See [agentskills.io](https://agentskills.io/home) for platform-specific instructions.

## Used with Carbone MCP

The Carbone Skill pairs with the [Carbone MCP server](https://carbone.io/documentation/developer/ai/mcp.html). The skill gives the AI assistant deep knowledge of Carbone's templating syntax, while the MCP server provides direct API access to render documents. Together, they let you design templates and generate documents in a single conversation: the skill handles syntax correctness, the MCP handles API actions.

## What the skill covers

- Tag syntax: `{d.}`, `{c.}`, aliases, i18n, runtime options
- Formatters: date, number, string, currency, and chaining
- Conditions: `showBegin/showEnd`, `hideBegin/hideEnd`, ternary expressions
- Loops: arrays, nested loops, aggregation, drop-row patterns
- `:set` patterns for computed and conditional values
- HTML templates: CSS injection, charts, headers/footers, PDF options
- Markdown templates: loops in tables, converting to DOCX/PDF
- Embedding Carbone tags into AI-generated DOCX, XLSX, and PPTX files
- Best practices and a validation checklist
- Template formats: `XML` `HTML` `ODT` `ODS` `ODP` `DOCX` `XLSX` `PPTX` `ODG` `SVG` `MD` `IDML` — and `PDF` form templates
- Export formats: `PDF` `DOCX` `XLSX` `PPTX` `ODT` `ODS` `ODP` `HTML` `CSV` `TXT` `JPG` `PNG` `GIF` `SVG` `WEBP` `EPUB` `MD` `RTF` (160+ conversions)

## How to update

Re-download the latest release when Carbone ships major updates:

**[Latest release](https://github.com/carboneio/carbone-skill/releases/latest)**

Changelog: [carbone.io/changelog.html](https://carbone.io/changelog.html)

## Contributing

Spotted an error or missing formatter? See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Apache-2.0
