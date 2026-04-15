---
name: carbone
description: >
  Use this skill when the user:
  - Asks how to write a Carbone tag, loop, condition, or formatter
  - Wants to build or design a template (invoice, report, contract, spreadsheet, presentation)
  - Asks to validate, fix, or explain Carbone syntax, or map JSON data into a template
  - Mentions Carbone, {d.}, {c.}, formatters, or document generation from JSON
  - Uses terms like "Carbone tag", "Carbone placeholder", "Carbone field", "document placeholder", "{d.", "{c.", "fill a template with data", "generate a report from data", "Markdown to PDF", "HTML to PDF"
  - Asks about HTML templates (CSS injection, charts, headers/footers, PDF options) or Markdown templates (loops in tables, DOCX/PDF output)
  - Is generating a DOCX, XLSX, or PPTX and needs to embed Carbone tags
  - Wants to generate any document (PDF, Word, Excel, PowerPoint, HTML, Markdown) from JSON data — even if they don't mention "Carbone" by name
  Always consult this skill before answering Carbone questions. NEVER invent syntax.
license: Apache-2.0
metadata:
  author: carboneio
  version: "1.1.1"
  repository: https://github.com/carboneio/carbone-skill
---

# Carbone Universal Templating Language — Skill

Carbone is a **declarative** templating engine. You provide a template (DOCX, XLSX, PPTX, ODT, HTML, …) with **Carbone tags** (placeholders in `{}`), plus a JSON dataset. Carbone merges both to produce the filled document.

**Important**: Never invent syntax. Only use what is documented here or in `references/formatters.md`.

---

## 1. Tag Types and Prefixes

| Tag | Purpose |
|---|---|
| `{d.path}` | Data from the main JSON object (most common) |
| `{c.path}` | Data from the "complement" object (metadata / computed values) |
| `{c.now}` | Built-in: current date/time at render time (UTC). Chain with `:formatD` to display it. |
| `{#alias = d.path}` | Declare an alias shortcut (removed from output) |
| `{$alias}` | Use a declared alias |
| `{t(key)}` | Static i18n translation tag — no quotes needed, whitespace preserved |
| `{o.option=value}` | Carbone runtime option (removed from output) |

### Special `{o.}` options
Common: `{o.timezone=Europe/Paris}` `{o.lang=en-US}` `{o.converter=L}` `{o.useHighPrecisionArithmetic=true}` `{o.exportFormattedValuesAsText=true}` (XLSX only) `{o.preReleaseFeatureIn=VERSION}` (opt-in pre-release features).
Full option list → `references/runtime-options.md`.

---

## 2. Basic Placeholders

| Goal | Tag |
|---|---|
| Simple property | `{d.firstname}` |
| Nested property | `{d.user.address.city}` |
| Array item by position | `{d.items[0].name}` or `{d.items[i=0].name}` |
| Last array item | `{d.items[i=-1].name}` |
| Second to last | `{d.items[i=-2].name}` |
| Parent object property | `{d.child..parentProp}` (`..` = up 1 level, `...` = up 2 levels) |

### Array search (without loop)
Search the first matching item by attribute value:
```
{d[year=1999].name}
{d[name='Back to the future'].year}
{d[year=1999, meta.type='SF'].name}     ← multiple filters = AND
{d.subArray[sub.b.c = .other].text}     ← variable filter (starts with .)
{d.movies[year=1999]..country}          ← access parent via ..
```
If the filter returns multiple rows, Carbone keeps only the **first** match.

---

## 3. Aliases

```
{#myAlias = d.wheels}
{d.name} needs {$myAlias} wheels.
```

**Parametrized aliases** (act like functions):
```
{#mealOf($weekday) = d[weekday = $weekday]}
Tuesday we eat {$mealOf(2).name}.
```

Aliases are shortcuts — they do NOT store values. To store a computed value, use `:set` (see Section 7).

---

## 4. Loops (Repetitions)

Carbone uses a **two-row declaration** pattern. No explicit open/close tags needed.

### 4a. Simple array loop
```
{d.cars[i].brand}     {d.cars[i].id}
{d.cars[i+1].brand}
```
The `[i+1]` row marks the end of the repeated block and is **removed** from output.

### 4b. Nested loops (unlimited depth)
```
{d[i].brand}
{d[i].models[i].size}   {d[i].models[i].power}
{d[i].models[i+1].size}
{d[i+1].brand}
```

### 4c. Loop over object properties
Use `.att` to print the key, `.val` to print the value:
```
{d.myObject[i].att}    {d.myObject[i].val}
{d.myObject[i+1].att}
```

You can also combine **direct property access and object iteration on the same object** in the same template (v5.4.2+, requires `{o.preReleaseFeatureIn=5004002}`):
```
{d.myObj.id}
{d.myObj[i].att}
{d.myObj[i+1].att}
```

### 4d. Filtering
Operators: `>`, `<`, `>=`, `<=`, `=`, `!=`. Place conditions after the iterator:
```
{d[i, age > 19, age < 30].name}
{d[i+1, age > 19, age < 30].name}
```
**v5+**: only the `[i+1]` row needs the filter. First N items: `{d[i, i < 2].name}`. Last item only: `{d[i=-1].name}`. Exclude last: `{d[i, i!=-1].name}`.

For negative-index patterns, OR-logic with `:set`, and advanced filter examples → read `references/loops-advanced.md`.

### 4e. Grouping (v5+)
Use an attribute name as a custom iterator to group rows by that attribute:
```
{d[brand].brand}   {d[brand].qty:aggSum(.brand)}
{d[brand+1].brand}
```

For advanced loop patterns (bidirectional loops, sorting, distinct, lookup/JOIN, parallel loops, loops on primitive arrays, loops on nested arrays, object attribute search, row repetition) → read `references/loops-advanced.md`.

---

## 5. Formatters

Formatters are chained with `:`. Each formatter's output feeds the next.
```
{d.name:lowerCase:ucFirst}
{d.birthday:formatD(LL)}
```
Parameters use parentheses. Strings with spaces or commas need single quotes: `:prepend('hello world')`.

**Escaping a single quote**: use two adjacent single quotes — `'David''s Car'`.

**`{t()}` in formatter args**: `{d.id:ifEQ(2):show( {t(monday)} ):elseShow( {t(tuesday)} )}`. No quotes needed around `{t()}` content — whitespace is preserved. Chain formatters after `:show`: `{d:show('{t(Destination)}'):ucFirst}`.

**Whitespace in JSON key names** (requires `{o.preReleaseFeatureIn=4022011}`): wrap in single quotes — `{d.'my param'.'second level'[1].'sub obj'}`. Not supported inside formatters, array filters, or aliases.

**Dynamic parameters** start with `.` (relative) or `d.`/`c.` (absolute): `{d.qtyB:add(.qtyC)}` / `{d.qtyB:add(d.qtyA)}`.

For the **complete formatter reference**, read `references/formatters.md`.

### Quick-reference — most common formatters

**Text**: `:lowerCase` `:upperCase` `:ucFirst` `:ucWords` `:prepend(text)` `:append(text)` `:replace(old,new)` `:substr(start,end)` `:ellipsis(max)` `:convCRLF` `:html` `:print(msg)` `:t`

**Number**: `:formatN(precision)` (uses `lang` option for separators) `:add(x)` `:sub(x)` `:mul(x)` `:div(x)` `:mod(x)` `:abs` `:round(n)` `:ceil` `:floor`

**Math formula** — formatters accept simple expressions (no parentheses, `*`/`/` before `+`/`-`):
```
{d.val:add(.otherQty + .vat * d.sub.price - 10 / 2)}
```

**Currency**: `:formatC(precisionOrFormat, targetCurrency)` `:convCurr(target, source)`

**Date**: `:formatD(patternOut, patternIn)` `:addD(amount, unit)` `:subD(amount, unit)` `:startOfD(unit)` `:endOfD(unit)` `:diffD(toDate, unit, patternFrom, patternTo)`

**Interval**: `:formatI(patternOut, patternIn)` — converts ms/seconds/ISO durations to human-readable

**Array**: `:arrayJoin(sep, index, count)` `:arrayMap(objSep, attSep, attributes)` `:count(start)`

**Key-value mapping**: `:convEnum('ENUM_NAME')` — enum defined in render options

---

## 6. Conditions

### 6a. Inline — print a word/phrase
```
{d.score:ifGT(50):show('Pass'):elseShow('Fail')}
```
Switch-case pattern:
```
{d.val:ifEQ(1):show(A):ifEQ(2):show(B):elseShow(C)}
```

### 6b. Conditional blocks — show/hide a section
```
{d.isPremium:ifEQ(true):showBegin}
  ... premium content ...
{d.isPremium:showEnd}

{d.status:ifEQ('cancelled'):hideBegin}
  ... content hidden if cancelled ...
{d.status:hideEnd}
```
Recommendation: use only line breaks (`Shift+Enter`) between Begin/End tags to avoid extra blank lines.

### 6c. Smart conditions — drop/keep elements (ENTERPRISE, v4+)
Drops the containing document element when condition is true. The tag prints **nothing**.
```
{d.discount:ifEM():drop('row')}
```
Elements: `row` `p` `img` `table` `chart` `shape` `slide` (ODP) `item` (ODP/ODT) `sheet` (ODS) `h` (ODT).
Drop N consecutive: `{d.text:ifEM():drop(row, 3)}`. Use `:keep` for the inverse.
Format limits: ODS → `row`/`img` only. HTML → `table`/`row`/`p`. XLSX → `row` only.

### 6d. Logical operators
`:ifEQ(v)` `:ifNE(v)` `:ifGT(v)` `:ifGTE(v)` `:ifLT(v)` `:ifLTE(v)` `:ifIN(v)` `:ifNIN(v)` `:ifEM()` `:ifNEM()` `:ifTE(type)`

Chain with `:and(.prop)` or `:or(.prop)` (OR is the default between consecutive conditions):
```
{d.age:ifGTE(18):and(.country):ifEQ('FR'):show('Eligible'):elseShow('Not eligible')}
```

---

## 7. Computation

### 7a. Simple mathematics
`:add`, `:sub`, `:mul`, `:div`, `:mod`, `:abs` — see formatters reference.
High-precision mode: add `{o.useHighPrecisionArithmetic=true}` to the template.

### 7b. Aggregators (ENTERPRISE, v4+)
Process an entire array and return a single result. Place **outside** the loop for totals:
```
{d.cars[].qty:aggSum}                ← sum of all qty
{d.cars[sort>1].qty:aggSum}          ← sum with filter
{d.cars[].qty:mul(.sort):aggSum}     ← with chained formatter before agg
```
Inside a loop, partitioned by group:
```
{d[i].qty:aggSum(.brand)}    ← sub-total per brand
```
All aggregators: `aggSum` `aggAvg` `aggMin` `aggMax` `aggCount` `aggCountD` `aggStr(sep)` `aggStrD(sep)` `cumSum` `cumCount` `cumCountD`

Limitation: to aggregate a value from a sub-object, use `:print` first:
```
{d[].qty:print(.sub.price):aggSum}
```

`:aggSum` also works when the root `d` is an array with nested loops (v5.4.2+):
```
{d[].sub[].val:aggSum}
```

### 7c. Store and Transform — `:set` (ENTERPRISE, v5+ / v4 with `{o.preReleaseFeatureIn=4022011}`)

Store into `c.`: `{d.cars[].qty:aggSum:set(c.mySum)}` / `Total: {c.mySum}`
Add attribute to items: `{d.cars[].qty:append(' tyres'):set(.newInfo)}`
Clone array: `{d.myArray[]:set(c.new[])}` / `{d.myArray[].country:set(c.newArr[].country)}`
Distinct merge into nested: `{d[].city:set(c.countries[id=.country].cities[].name)}`

**Rules**: tags using `:set` print nothing. Use `c.` for new data. Alphanumeric keys only. `[i]` cannot be used with `:set`.

For complex `:set` patterns (dynamic URLs, type conversion, newest date from list, multi-aggregation conditions, filtering parent by child content, paginating into N-column rows, JSON restructuring, grouping flat lists, pairing sibling arrays, dynamic object keys) → read `references/set-patterns.md`.

---

## 8. Advanced Features

### 8a. Dynamic pictures (ENTERPRISE, v3+)
Insert a placeholder image in the template, then write the Carbone tag in the image's **alternative text** (not inline in text):
```
{d.imageUrl}
```
Supports public URLs and Base64 Data URIs. Compatible with PDF, ODT, ODS, ODP, ODG, PPTX, XLSX, DOCX.

**`:imageFit(option)`** — controls how the replacement image is resized to fit the placeholder (DOCX/ODT only):
- `fillWidth` (default) — fills the full width while keeping aspect ratio
- `contain` — fits within the placeholder box while keeping aspect ratio
- `fill` — stretches to fill the entire placeholder box

```
{d.myImage:imageFit(contain)}
```

### 8b. Barcodes (ENTERPRISE, v3.4.6+)
Insert a placeholder image, write in its **alternative text**:
```
{d.code:barcode(qrcode)}
{d.code:barcode(code128)}
{d.code:barcode(qrcode, svg:true)}              ← vector SVG, better print quality
{d.code:barcode(ean13, width:200, height:100)}  ← custom dimensions (mm)
```
Common barcode options (second argument, `key:value` format): `width`, `height`, `scale` (1–10), `includetext` (true/false), `textsize`, `textxalign` (left/center/right/justify), `textyalign` (below/center/above), `rotate` (N/R/L/I), `barcolor` (#RRGGBB), `textcolor` (#RRGGBB), `backgroundcolor` (#RRGGBB), `eclevel` (L/M/Q/H — QR only).
Carbone supports 107 barcode types.

### 8c. Colors (ENTERPRISE, v4.17+)
`:color(scope, type)` — injects color from data into the document element. The tag prints nothing. Color must be 6-digit hex (e.g. `FF0000` or `#FF0000`). Scopes: `p`, `row`, `cell`, `shape`, `part`. Types: `text` (default), `highlight`, `background`, `border`.
```
{d.statusColor:color(row, background)}
{d.textHex:ifEQ('ok'):show(007700):elseShow(FF0000):color(p)}
```
For full scope/type tables, limitations, and `:bindColor` (legacy) → read `references/advanced-features.md`.

### 8d. HTML content rendering (ENTERPRISE, v5+)
`:html` converts an HTML string to native document formatting (ODT/DOCX/PDF):
```
{d.richContent:html}
{d.notes:convCRLF:html}    ← convert \n to <br> first
```
Options: `html(inline)`, `html(nospace)`, `html(tabletheme:Name)` (DOCX), `html(headingtheme:prefix-)` (DOCX). Page breaks via `style="break-before:page"` (requires `{o.preReleaseFeatureIn=5002000}`). Full options → `references/advanced-features.md`.

### 8e. Hyperlinks (ENTERPRISE, v3+)
Set the URL of a hyperlink (in your editor) to a Carbone tag, e.g. `{d.url}`. Works in DOCX, XLSX, PPTX, ODT, ODS, ODP, ODG. Use `:defaultURL('fallback')` if the URL may be invalid. Edge cases (XLSX syntax, mixed URLs) → `references/advanced-features.md`.

### 8f. i18n / Translations
- Static text: `{t('key')}` in template + localization dictionary JSON
- Dynamic value: `{d.status:t}` — looks up the value in the translation dictionary
- Use locale-aware formatters for numbers/dates/currencies: `:formatN`, `:formatD`, `:formatC`, `:formatI`

### 8g. Key-value mapping
`:convEnum('ENUM_NAME')` — maps index or key to a human-readable label.
`ENUM_NAME` is defined in the `enum` property of render options.

### 8h. File operations (ENTERPRISE, v4.22+)
Append PDF files at the end (or start) of the generated PDF:
```
{d.products[i].datasheet:appendFile}           ← append at end (default)
{d.products[i].datasheet:appendFile('start')}  ← append at start
```
Attach a file inside a PDF (Factur-X, ZUGFeRD):
```
{d.xmlUrl:attachFile('invoice.xml', 'text/xml')}
```

**`:appendTemplate(templateIdOrVersionId, position?)`** (v5.0.3+) — generate another stored template with the current data and append its PDF output. Each iterated array item becomes the root `{d.}` of the nested template. All render options are forwarded automatically. The tag prints nothing.
```
{d.myArray[i]:appendTemplate(110212)}
{d.myArray[i+1]}
```

### 8i. Digital signatures (ENTERPRISE, BETA — v5+)
Places a signature field. Tag prints nothing; returns position coordinates in the API response:
```
{d.signatureBuyer:sign}
```

### 8j. Transform / move shapes (ENTERPRISE, v4.22.11+ — requires `{o.preReleaseFeatureIn=4022011}`)
Move shapes/images on X or Y axis in ODP/PPTX/ODT:
```
{d.offset:transform('x', 'cm')}
```
Units: `cm`, `mm`, `inch`, `pt`. Write inside the shape's alt text or inside the shape itself.

### 8k. SVG templates
SVG files can be used as Carbone input templates. Write tags inside HTML comments (`<!-- {d.value:svgUpdate(...)} -->`). Use `:svgUpdate(attrName, selectorValue, selectorType)` to update attributes, or `:svgSelectiveUpdate` to select and inject. Full syntax → `references/advanced-features.md`.

### 8l. Dynamic Forms — ODT Text Fields and Checkboxes (ENTERPRISE, v3.3+)
ODT editable text fields and clickable checkboxes (LibreOffice only). Checkbox alternatives that work in any format: `{d.value:ifEQ(true):show(☑):elseShow(☐)}` (unicode) or emoji. Full setup steps → `references/advanced-features.md`.

### 8m. Native Charts (ENTERPRISE, v4+)
Three chart types: native DOCX charts (loop tags in the embedded Excel sheet), ODT/LibreOffice charts (`{bindChart(refValue) = d.tag}` in the Data Table), and ECharts via `:chart` formatter in the alt text of a placeholder image (all formats). For full syntax and examples → read `references/advanced-features.md`.

### 8n. PDF form filling (ENTERPRISE, v5.0.0-beta.11+)
Three methods to fill PDF form fields:

**Method 1** — Place Carbone tags directly in text fields of a PDF form. Loops are allowed.

**Method 2** — Add annotations above form fields with these formatters:
```
{d.myText:fill}                        ← fills text field behind the tag
{d.myCondition:ifEQ(true):check}       ← checks checkbox/radio behind the tag
{d.myCondition:ifEQ(false):uncheck}    ← unchecks checkbox behind the tag
```

**Method 3** — Target fields by name anywhere in the PDF:
```
{d.text:fillField('fieldName')}                              ← fill text field or check checkbox
{d.genre:ifEQ('boy'):show(male):fillField('radioGroup')}    ← select radio button option
{d.confirm:ifEQ(true):checkField('fieldName')}              ← check checkbox if condition true
```
Supported field types: Text, Radio buttons, Checkboxes.

---

## 9. Design Best Practices

### Editor configuration (critical!)
Text editors auto-replace straight quotes `'` with "smart quotes" — **Carbone only accepts straight single quotes** in formatter parameters.

- **MS Word**: `Tools > AutoCorrect Options > AutoFormat As You Type` → uncheck "Straight quotes with smart quotes"
- **LibreOffice**: `Tools > AutoCorrect > Localized Options` → disable Replace for single and double quotes

### Tag styling
Only the font style and size of the **first `{` character** applies to the output. You can make the rest of the tag tiny (e.g. size 1pt) to reduce visual clutter in the template.

### Template type choice
- **PDF with high styling control**: use HTML — full CSS, custom paper size, margins, headers/footers via `<carbone-pdf-options>`
- **PDF from a document template**: use DOCX or ODT — better pagination, text overflow handling
- **Presentations**: PPTX uses absolute positioning — loops don't auto-create new slides; paginate manually with filters. ODP (LibreOffice) supports dynamic slide creation
- **Spreadsheets**: formulas recalculate after Carbone injection; use `:formatN` so tags output native numbers not strings

### XLSX/ODS — formula rules
Carbone tags and Excel formulas must be in **separate cells**. For totals after loop injection, prefer Carbone aggregators: `{d.items[].qty:aggSum:formatN}`. For `INDIRECT`+`ROW` and `MATCH`/`INDEX` patterns → read `references/xlsx-tips.md`.

### Removing the trailing comma/separator in a loop
```
{d.list[i, i=-1]:ifNEM:drop(p)}    ← drop last item's paragraph (e.g. trailing comma)
```
Or use `:arrayJoin` to flatten arrays without a loop when you just need a string.

### Prefer `:drop` over `hideBegin/hideEnd`
`:drop` is simpler, cleaner, and removes the element without leaving empty space. Use `hideBegin/hideEnd` only when you need to hide large multi-element blocks.

### Floating elements in loops
Avoid floating images, shapes, or text boxes inside Carbone loops. Set their anchor to **"In line with text"** to prevent invalid output.

### `{bindColor}` pitfall with loops
If the same reference color is used in **multiple places or multiple loops** in the template, Carbone may inject the wrong value into the wrong location and corrupt the document. Every reference color used with `{bindColor}` must appear in **exactly one location** in the template.

### Deprecated formatters — use modern equivalents instead
The following formatters still work but should be avoided in new templates:

| Deprecated | Modern equivalent |
|---|---|
| `:ifEmpty(text)` | `:ifEM():show(text)` |
| `:ifEqual(val, text)` | `:ifEQ(val):show(text)` |
| `:ifContain(val, text)` | `:ifIN(val):show(text)` |
| `:convDate(patternIn, patternOut)` | `:formatD(patternOut, patternIn)` |
| `:int` | `:round(0)` or `:floor` |
| `:toFixed` | `:round(n)` |
| `:toEN` | `:formatN()` with `lang:"en-us"` |
| `:toFR` | `:formatN()` with `lang:"fr-fr"` |

### MS Word — table column sizing
**Do not** resize table columns by mouse-dragging — it creates imprecise widths. Instead: right-click column → **Table Properties → Column → set "Preferred width"** to an exact value.

---

## 10. Validation Checklist

When asked to validate a Carbone tag, check:

1. **Braces** — `{` and `}`, NOT `{{` (that's Mustache, not Carbone)
2. **Correct prefix** — `d.`, `c.`, `#`, `$`, `t(`, `o.`
3. **No spaces in tag name** — `{d.first name}` is invalid → use `{d.firstName}`
4. **Dot notation** — properties separated by `.`, arrays with `[index]`
5. **Formatter separator** — `:` not `.` or `|`
6. **Formatter params** — inside `()`, strings with spaces/commas need single quotes `'text'`
7. **Formatters must be chained, not nested** — `{d.value:add(20):sub(10)}` ✅ — `{d.value:add(20:sub(10))}` ❌ (nesting doesn't work)
8. **Loop end-marker** — every `[i]` block must have its `[i+1]` end-marker row
9. **`showBegin/showEnd` and `hideBegin/hideEnd` must always be paired** — a missing `showEnd` will break the document
10. **Never interleave two loops** — this corrupts the document:
    ```
    ❌ {d.fruits[i].name} {d.vegetables[i].name} {d.fruits[i+1].name}
       {d.vegetables[i+1].name}
    ```
    Each loop's `[i]` and `[i+1]` must stay together as a complete block:
    ```
    ✅ {d.fruits[i].name} {d.fruits[i+1].name}
       {d.vegetables[i].name} {d.vegetables[i+1].name}
    ```
    If you need to **access data from multiple different arrays within the same loop row**, use **Lookups** (section 4j). Lookups let you reach into any other array from inside a single loop — there is no limit on how many different arrays you can access this way, and no risk of interleaving corruption.
11. **Sorting** — uses attribute name as iterator (e.g. `[power, i]` / `[power+1, i+1]`), NOT `sortAsc()`/`sortDesc()` functions
12. **Filtering** — uses `[i, prop > value]` directly in brackets, NOT a `where()` function
13. **Grouping** — uses attribute name `[brand]`/`[brand+1]`, NOT a `groupBy()` function
14. **Condition chaining** — a logical operator (`:ifEQ` etc.) must be followed by `:show`, `:elseShow`, `:hideBegin/End`, `:showBegin/End`, or `:drop`/`:keep`
15. **`:drop` / `:keep` / `:set` / `:sign`** — these all print **nothing** in the output
16. **Dynamic images** — must be a full Data-URI (`data:image/jpeg;base64,...`) or a public URL — pure base64 without the prefix is not supported

---

## Reference files

- `references/formatters.md` — complete list of all formatters with parameters and examples
- `references/loops-advanced.md` — bidirectional loops, sorting, distinct, lookup/JOIN, parallel loops, loops on primitive/nested arrays, object attribute search, row repetition
- `references/set-patterns.md` — advanced `:set` patterns: dynamic URLs, type conversion, date aggregation, JSON restructuring, flat list grouping, N-column pagination, sibling array pairing, dynamic object keys
- `references/html-templates.md` — HTML-specific features: inline CSS injection, `:chart`, barcodes in `<img src>`, `<carbone-pdf-header/footer>`, page breaks, `<carbone-pdf-options>`, PDF conversion API
- `references/markdown-templates.md` — Markdown-specific features: table loops, limitations (no barcodes, no page breaks, no HTML injection yet), convert to PDF/DOCX/ODT/PNG/JPG
- `references/runtime-options.md` — All `{o.}` template options with descriptions and version requirements
- `references/advanced-features.md` — Full `:color` reference, `:html` options and page breaks, hyperlink edge cases, SVG templates, ODT dynamic forms, native charts (DOCX, ODT `bindChart`, ECharts)
- `references/xlsx-tips.md` — How to compute totals in Excel/ODS spreadsheets when Carbone injects new rows via loops: dynamic `INDIRECT`+`ROW` formulas, Carbone aggregators, `MATCH`/`INDEX` referencing

**When to read which reference:**
- User asks about HTML template, CSS injection, chart formatter, `<carbone-pdf-options>`, headers/footers → read `references/html-templates.md`
- User asks about Markdown template, MD loops, converting MD to PDF/DOCX → read `references/markdown-templates.md`
- User asks about bidirectional loops, sorting, lookup, parallel loops, arrays of primitives → read `references/loops-advanced.md`
- User needs complex data transformation, JSON restructuring, aggregation patterns, or multi-step `:set` logic → read `references/set-patterns.md`
- User asks about `{o.}` template options, timezone, lang, converter, pre-release flags → read `references/runtime-options.md`
- User asks about `:color` scopes/types, `:html` options, hyperlink edge cases, SVG templates, ODT forms, or native chart syntax → read `references/advanced-features.md`
- User asks how to compute totals or use Excel formulas in a spreadsheet where Carbone injects rows → read `references/xlsx-tips.md`

Official docs: https://carbone.io/documentation/design/overview/getting-started.html
