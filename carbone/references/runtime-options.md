# Carbone Runtime Options — `{o.}` Reference

Read this file when the user asks about runtime options (`{o.}` tags): timezone, language/locale, converter settings, complement data, hard refresh, pre-release feature flags, or any `{o.}` option.

Runtime options are placed anywhere in the template — they are removed from output and apply globally to the render.

| Option | Description |
|---|---|
| `{o.useHighPrecisionArithmetic=true}` | Enables arbitrary-precision decimal arithmetic (v4.22.4+) |
| `{o.hardRefresh=true}` | Forces converter processing even when input and output format are the same — useful to refresh XLSX formulas after data injection (v5.4.4+) |
| `{o.preReleaseFeatureIn=VERSION}` | Activates a pre-release feature by version number (e.g. `5002000`) |
| `{o.timezone=Europe/Paris}` | Forces the timezone used by date formatters; overrides the API option (v5.4.3+) |
| `{o.lang=en-US}` | Forces the language/locale; overrides the API option. Accepts upper or lowercase (v5.4.3+) |
| `{o.converter=L}` | Forces the document converter: `L` = LibreOffice, `O` = OnlyOffice, `C` = Chrome (v5.4.3+) |
| `{o.exportFormattedValuesAsText=true}` | Forces `:formatN` to output localized text strings instead of native XLSX number format. **XLSX templates only** (v5.4.3+) |
| `{o.preReleaseFeatureIn=4025011}` | Activates **all** v4 pre-release features up to v4.25.11 — use when you need any v4 pre-release feature without specifying the exact version code |
| `{o.preReleaseFeatureIn=5004002}` | Enables fix for combining direct object access with object iteration on the same object (v5.4.2, opt-in only) |
| `{o.styleSource=templateOrVersionId}` | Apply style from another DOCX/ODT template to a Markdown template. The style template can itself contain Carbone tags. Styles applied: headings, tables, headers, footers. Effective only when converting Markdown to DOCX, ODT, or PDF (v5.8.0+) |
