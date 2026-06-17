# Carbone Advanced Features Reference

Read this file when the user asks for full details on: the `:color` formatter (scopes, types, limitations, `:bindColor`), the `:html` formatter (full reference — supported elements, CSS, options, page breaks, entities, common patterns), native charts (DOCX, ODT/LibreOffice `bindChart`, ECharts), hyperlink edge cases, SVG templates, ODT dynamic forms, or removing placeholder images when value is empty.

---

## Colors — full reference (ENTERPRISE, v4.17+)

`:color(scope, type)` — the tag prints nothing in the document. Color must be **6-digit hex**, with or without `#` (e.g. `FF0000` or `#FF0000`). Invalid colors are replaced with light gray `#888888`.

| `scope` | Element targeted |
|---|---|
| `p` or `paragraph` | Current paragraph (default) |
| `row` | Current table row |
| `cell` | Current table cell |
| `shape` | Current shape |
| `part` | Inline section within a paragraph (ODT only) |

| `type` | What changes |
|---|---|
| `text` | Text color (default) |
| `highlight` | Text highlight color (**not supported in DOCX**) |
| `background` | Background of cell, row, or shape |
| `border` | Border of shape only |

```
{d.statusColor:color(row, background)}
{d.textHex:ifEQ('ok'):show(007700):elseShow(FF0000):color(p)}
```

**Known limitations:**
- Cannot be used in aliases or combined with aggregators
- PPTX (v5.6.0+): all combinations of `scope` (row, cell, shape) and `type` (text, background, border) are supported; `border` applies to shapes only
- For ODP (text, tables, shapes) and ODT (shapes only): a non-default style must already be applied to the target element in the template
- Complex nested tables with colors on sub-tables are not fully supported

**`:bindColor` (old method — use `:color` instead)**
Static color replacement in template: `{bindColor(myColorRef, format) = d.myVar}`
- `myColorRef` is the placeholder color already applied in the template (used to identify what to replace)
- `format`: `#hexa`, `hexa`, `color` (named), `rgb` (object), `hsl` (object)
- For DOCX text **background**: only 17 named colors work (`yellow`, `green`, `cyan`, `magenta`, `blue`, `red`, `darkBlue`, `darkCyan`, `darkGreen`, `darkMagenta`, `darkRed`, `darkYellow`, `darkGray`, `lightGray`, `black`, `white`, `transparent`)

---

## Native Charts (ENTERPRISE, v4+)

### DOCX/Word
Insert a chart via `Insert > Chart`, edit the embedded Excel spreadsheet with Carbone loop tags directly in cell values:
```
{d.temps[i].date}   {d.temps[i].min}   {d.temps[i].max}
{d.temps[i+1].date}
```
⚠️ Each `[i]` expression must have its own `[i+1]` on the next row — you cannot use a single `[i+1]` for multiple `[i]` tags.

Aliases work inside chart cells too — declare `{#income = d.income_type_summary}` in the document body, then loop the alias array:
```
{$income[i].itg_nm}
{$income[i+1].itg_nm}
```

### ODT/LibreOffice
Insert a chart, edit the Data Table. Use `{bindChart(refValue) = d.tag}` in the Categories column to bind a JSON value to a chart reference value:
```
{d.cheeses[i].type} {bindChart(3)=d.cheeses[i].purchasedTonnes}
{d.cheeses[i+1].type} {bindChart(4)=d.cheeses[i+1].purchasedTonnes}
```
`bindChart(3)` means: replace the value `3` in the chart with the result of the tag.

### ECharts (all formats)
Insert a placeholder image, write `:chart` tag in its alt text:
```
{d.chartOptions:chart}
```
JSON must contain a full ECharts v5 config:
```json
{ "type": "echarts@v5a", "width": 600, "height": 400, "theme": null, "option": { /* ECharts config */ } }
```
Compatible with: ODT, ODS, ODP, ODG, DOCX, PPTX, XLSX, HTML, PDF. Use `echarts@v5a` (not `echarts@v5`) for better SVG compatibility.

**Chart from an indexed array** — when data contains an array of ECharts config objects, access each by index and chain `:imageFit` to fit the chart to the placeholder size:
```
{d.team_avg_chart[0]:chart:imageFit(contain)}
{d.team_avg_chart[1]:chart:imageFit(contain)}
{d.team_avg_chart[2]:chart:imageFit(contain)}
```
Each tag goes in a separate image placeholder alt-text. The number of placeholders in the template determines how many charts render.

---

## `:html` Formatter — Full Reference (ENTERPRISE, v5+)

`:html` converts an HTML string into native document formatting. Use it whenever JSON data contains an HTML payload that must render as styled content rather than escaped text. Without `:html`, the `<` and `>` characters are escaped and the raw HTML is shown as text.

```
{d.richContent:html}
{d.notes:convCRLF:html}    ← convert \r\n / \n to native line breaks first; with :html, \n becomes <br> (v3.5.3+)
```

**Compatible templates**: ODT, DOCX, HTML (PDF is reached via conversion from these — PDF is not a Carbone template format).

### Supported HTML elements

| Category | Elements | Notes |
|---|---|---|
| Text formatting | `<b>` `<strong>` `<i>` `<em>` `<u>` `<s>` `<del>` | Always supported |
| Structure | `<p>` `<br>` | Always supported |
| Lists | `<ul>` `<ol>` `<li>` | Always supported |
| Links | `<a href="...">` | URL validated; use `:defaultURL` for fallback |
| Images | `<img src="..." width="X" height="Y">` | URL or Data-URI; size in px; default 5cm if missing |
| Headings | `<h1>` `<h2>` `<h3>` `<h4>` `<h5>` `<h6>` | Requires `{o.preReleaseFeatureIn=5002000}` |
| Tables | `<table>` `<tr>` `<th>` `<td>` with `colspan`/`rowspan` | Requires `{o.preReleaseFeatureIn=5002000}` (introduced in v5.0.9, significant fixes in v5.2.0 — use 5002000) |

**Not supported** (ignored): `<tbody>` `<thead>` `<caption>`, external stylesheets, `<style>` elements, CSS classes, CSS IDs.

### Supported CSS (inline `style` attribute only)

Support depends on the **template format**:

- **HTML template** → any inline CSS works, no restrictions (rendered by Chromium)
- **DOCX or ODT template** → only the following properties are supported:

| Property | Values |
|---|---|
| `color` | text color |
| `background-color` | background color |
| `break-before` | `page` — insert page break before `<p>` (requires `{o.preReleaseFeatureIn=5002000}`) |
| `break-after` | `page` — insert page break after `<p>` (requires `{o.preReleaseFeatureIn=5002000}`) |

Supported color formats for DOCX/ODT: hex (`#FF00BB`), RGB (`rgb(31,120,50)`), HSL (`hsl(120,75,25)`), named CSS colors (`red`, `lightseagreen`, etc.).

Page-break example:
```html
<p style="break-before:page">New page starts here</p>
<p style="break-after:page">Page break after this</p>
```
Page breaks apply to `<p>` only — not allowed in headers, footers, or table cells.

### Options (combinable with comma)

| Option | Effect | Example |
|---|---|---|
| `inline` | Render inside current paragraph. Only: `<a>` `<b>` `<strong>` `<em>` `<i>` `<s>` `<del>` `<u>` | `{d.v:html(inline)}` |
| `nospace` | Remove empty paragraph added after block elements | `{d.v:html(nospace)}` |
| `tabletheme:Name` | Apply named Word table theme (DOCX only) | `{d.v:html(tabletheme:GridTable2-Accent3)}` |
| `headingtheme:prefix` | Use custom heading styles `prefix1`…`prefix6` (themes must already exist in the template) | `{d.v:html(headingtheme:my-theme-)}` |

Combine: `{d.value:html(nospace,headingtheme:my-theme-)}`

### Font and paragraph behaviour
- HTML content **inherits** font family and size from the template paragraph where the tag is placed
- Text alignment from the template is **not** retained
- By default, `:html` renders content in a **new paragraph**. Use `inline` to stay in the current paragraph
- Tab characters: use `&ensp;` or `&emsp;` — do NOT use `&#9;` (collapsed to space by HTML parsers)

### HTML entities (v5.5.0+)
The `:html` formatter supports named, decimal, and hexadecimal HTML entities:
- Named: `&amp;` `&copy;` `&nbsp;`
- Decimal: `&#169;` `&#128512;`
- Hexadecimal: `&#xA9;` `&#x1F600;`
- Emoji supported as literal Unicode or numeric entities

### Common patterns

**Wrap dynamic HTML content before rendering** — `:prepend` and `:append` operate on the raw string; `:html` then renders the assembled result. Order matters — reversing it would render HTML before the wrapper tags are added:
```
{d.htmlContent:prepend('<ul>'):append('</ul>'):html}
```

**Choose between HTML blocks via a condition** — `:show` / `:elseShow` select which raw HTML to render, then `:html` renders the chosen branch:
```
{d.displayNote:ifEQ(true):show(d.noteSection):elseShow(d.nothingSection):html}
{d.isAdmin:ifEQ(true):show(d.adminPanel):elseShow(d.userPanel):html}
{d.showComment:ifEQ(true):show(.comment):elseShow(''):html}
```

**Render plain-text line breaks as HTML breaks** — chain `:convCRLF` before `:html` so `\n` becomes `<br>`:
```
{d.notes:convCRLF:html}
```

**Fallback URL for embedded links** — place `:defaultURL` before `:html`:
```
{d.content:defaultURL(.urlOnError):html}
```

---

## Hyperlinks — edge cases (ENTERPRISE, v3+)

- If no protocol is provided, Carbone automatically prepends `https://`
- You cannot mix a static URL with a dynamic tag in the same link — `https://carbone.io{d.path}` won't work; Carbone keeps only the dynamic part
- **XLSX only**: do NOT wrap the tag in `{}`. Use `d.url` (not `{d.url}`) as the hyperlink URL

**`:defaultURL(url)`** — if the injected URL fails validation, use this fallback:
```
{d.documentUrl:defaultURL('https://carbone.io')}
{d.url:defaultURL(.urlOnError)}          ← dynamic fallback
{d.content:defaultURL(.urlOnError):html} ← place before :html
```

---

## SVG templates

SVG files can be used as Carbone input templates. Write Carbone tags inside HTML comments in the SVG file.

**`:svgUpdate(attrName, selectorValue, selectorType)`** — update an SVG attribute on elements matching a selector:
```
{d.myTable[].width:svgUpdate('width', .id)}
```
- `attrName`: attribute to update (e.g. `'fill'`, `'stroke'`, `'width'`)
- `selectorValue`: value used to find the target element
- `selectorType`: `'id'` (exact match), `'id > *'` (direct children), `'id *'` (all descendants, default)

**`:svgSelectiveUpdate(attrName, attrValue, selectorType)`** — select elements and inject a value:
```
{d.myTable[active=true].id:svgSelectiveUpdate('fill', .color)}
```
Only basic SVG shapes are updated: `rect`, `circle`, `line`, `ellipse`, `polygon`, `polyline`, `path`.

---

## ODT Dynamic Forms — Text Fields and Checkboxes (ENTERPRISE, v3.3+)

**ODT editable text fields** (LibreOffice only, outputs ODT or PDF):
- In LibreOffice: `Form > Text Box` → draw box → disable Design Mode → place `{d.value}` inside
- Not supported in DOCX, XLSX, ODS, or ODT files created with MS Word

**Clickable checkboxes** (ODT/LibreOffice only):
- `Form > Check Box` → right-click → Control Properties → put tag in the **Name** property
- Checkbox is ticked when value is `true`, non-empty string, non-empty array, or non-empty object

**Checkbox alternatives that work in any format:**
```
{d.value:ifEQ(true):show(✅):elseShow(⬜️)}    ← emojis
{d.value:ifEQ(true):show(☑):elseShow(☐)}       ← unicode
{d.value:ifEQ(true):show(.img1):elseShow(.img2)} ← SVG dynamic images
```

---

## Removing placeholder images when value is empty

When a dynamic picture URL is invalid, a barcode value is incorrect, or an ECharts config is missing or malformed, Carbone inserts a default SVG image — a square with an X (cross). To remove the placeholder entirely instead, add a second tag in the **same alternative text** using `:drop(img)`.

**Dynamic picture:**
```
{d.imageUrl}{d.imageUrl:ifEM:drop(img)}
```

**Barcode:**
```
{d.number:barcode(ean13)}{d.number:ifEM:drop(img)}
```

**ECharts:**
```
{d.chartOptions:chart}{d.chartOptions:ifEM:drop(img)}
```

The `:drop(img)` tag prints nothing — if the condition is true, it removes the entire placeholder image from the document. Both tags must be placed in the image's alternative text.
