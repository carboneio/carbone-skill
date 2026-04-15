# Carbone Advanced Features Reference

Read this file when the user asks for full details on: the `:color` formatter (scopes, types, limitations, `:bindColor`), `:html` rendering options and page breaks, native charts (DOCX, ODT/LibreOffice `bindChart`, ECharts), hyperlink edge cases, SVG templates, or ODT dynamic forms.

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

---

## HTML content rendering — full reference (ENTERPRISE, v5+)

`:html` converts an HTML string to native document formatting (ODT/DOCX/PDF):
```
{d.richContent:html}
{d.notes:convCRLF:html}    ← convert \n to <br> first
```

**`:html` options** (passed as argument):
- `{d.content:html(inline)}` — render HTML inline within the current paragraph (only `a`, `b`, `strong`, `em`, `i`, `s`, `del`, `u` tags supported)
- `{d.content:html(nospace)}` — render without adding extra empty paragraphs after `<p>` tags
- `{d.content:html(tabletheme:GridTable2-Accent3)}` — apply a specific Word table theme to injected tables (DOCX only)
- `{d.content:html(headingtheme:my-theme-)}` — apply a custom heading theme prefix (themes must be named `my-theme-1` through `my-theme-6` in the template)

**Page breaks inside `:html`** (requires `{o.preReleaseFeatureIn=5002000}`, applies to `<p>` only, not in headers/footers/table cells):
```html
<p style="break-before:page">New page starts here</p>
<p style="break-after:page">Page break after this</p>
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
