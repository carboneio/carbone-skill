# Carbone Runtime Options — `{o.}` Reference

Read this file when the user asks about runtime options (`{o.}` tags) available in Carbone templates.

Runtime options are placed anywhere in the template — they are removed from output and apply globally to the render.

| Option | Description |
|---|---|
| `{o.useHighPrecisionArithmetic=true}` | Enables arbitrary-precision decimal arithmetic (v4.22.4+) |
| `{o.preReleaseFeatureIn=VERSION}` | Activates a pre-release feature by version number (e.g. `5002000`) |
| `{o.timezone=Europe/Paris}` | Forces the timezone used by date formatters; overrides the API option (v5.4.3+) |
| `{o.lang=en-US}` | Forces the language/locale; overrides the API option. Accepts upper or lowercase (v5.4.3+) |
| `{o.converter=L}` | Forces the document converter: `L` = LibreOffice, `O` = OnlyOffice, `C` = Chrome (v5.4.3+) |
| `{o.exportFormattedValuesAsText=true}` | Forces `:formatN` to output localized text strings instead of native XLSX number format. **XLSX templates only** (v5.4.3+) |
| `{o.preReleaseFeatureIn=5004002}` | Enables fix for combining direct object access with object iteration on the same object (v5.4.2, opt-in only) |
