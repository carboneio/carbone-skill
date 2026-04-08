# Carbone Formatters — Complete Reference

All formatters below are verified from official Carbone documentation. Do NOT use any formatter not listed here.

## Contents

- [Text Formatters](#text-formatters)
- [Number Formatters](#number-formatters)
- [Currency Formatters](#currency-formatters)
- [Date Formatters](#date-formatters)
- [Interval Formatter](#interval-formatter)
- [Array Formatters](#array-formatters)
- [Aggregator Formatters](#aggregator-formatters-enterprise-v4)
- [Condition Formatters](#condition-formatters)
- [Store and Transform — `:set`](#store-and-transform--set-enterprise-v5--v4-with-prereleasefeaturein)
- [Key-value Mapping](#key-value-mapping)
- [Advanced / Misc Formatters](#advanced--misc-formatters)
- [`:html` Formatter — Full Reference](#html-formatter--full-reference-enterprise-v5)
- [`:color` Formatter — Full Reference](#color-formatter--full-reference-enterprise-v417)
- [PDF Form Formatters](#pdf-form-formatters)

---

## Text Formatters

| Formatter | Version | Description | Example |
|---|---|---|---|
| `:lowerCase` | v0.12.5+ | All lowercase | `'Hello':lowerCase` → `"hello"` |
| `:upperCase` | v0.12.5+ | All uppercase | `'Hello':upperCase` → `"HELLO"` |
| `:ucFirst` | v0.12.5+ | Capitalize first letter | `'hello world':ucFirst` → `"Hello world"` |
| `:ucWords` | v0.12.5+ | Capitalize each word | `'hello world':ucWords` → `"Hello World"` |
| `:print(message)` | v0.13.0+ | Always return `message` regardless of input | `null:print('N/A')` → `"N/A"` |
| `:printJSON` | v4.23.0+ | Stringify object/array to JSON string | `[1,2]:printJSON` → `"[1,2]"` |
| `:unaccent` | v1.1.0+ | Remove accents | `'crème':unaccent` → `"creme"` |
| `:convCRLF` | v4.1.0+ | Convert `\n`/`\r\n` to document line breaks | Supported in DOCX/PPTX/ODT/ODP/ODS |
| `:html` | v5.0+ ENTERPRISE | Render HTML string as native document formatting — [see full reference](#html-formatter--full-reference-enterprise-v5) | ODT/DOCX/PDF — use `:convCRLF:html` for newlines |
| `:substr(begin, end, wordMode)` | v4.18.0+ | Slice string | `'foobar':substr(0,3)` → `"foo"` |
| `:split(delimiter)` | v4.12.0+ | Split string into array | `'a,b,c':split(',')` → `["a","b","c"]` |
| `:padl(targetLength, padString)` | v3.0.0+ | Left-pad string | `'3':padl(4,'0')` → `"0003"` |
| `:padr(targetLength, padString)` | v3.0.0+ | Right-pad string | `'3':padr(4,'0')` → `"3000"` |
| `:ellipsis(maximum)` | v4.12.0+ | Truncate with `...` | `'hello world':ellipsis(5)` → `"hello..."` |
| `:prepend(text)` | v4.12.0+ | Add prefix | `'world':prepend('hello ')` → `"hello world"` |
| `:append(text)` | v4.12.0+ | Add suffix | `'hello':append(' world')` → `"hello world"` |
| `:replace(oldText, newText)` | v4.12.0+ | Find & replace all occurrences | `'foo bar':replace('foo','baz')` → `"baz bar"` |
| `:len` | v2.0.0+ | Length of string or array | `'hello':len` → `5` |
| `:t` | v4.23.2+ | Translate dynamic value via i18n dictionary | `{d.status:t}` |
| `:preserveCharRef` | v4.23.8+ | Preserve character references | — |

---

## Number Formatters

| Formatter | Version | Description | Example |
|---|---|---|---|
| `:formatN(precision)` | v1.2.0+ | Format number using `lang` option for separators | `1000.456:formatN()` with `lang:"en-us"` → `"1,000.456"` |
| `:round(precision)` | v1.2.0+ | Round correctly (not like toFixed) | `1.05:round(1)` → `1.1` |
| `:add(value)` | v1.2.0+ | Add two numbers | `1000.4:add(2)` → `1002.4` |
| `:sub(value)` | v1.2.0+ | Subtract | `1000.4:sub(2)` → `998.4` |
| `:mul(value)` | v1.2.0+ | Multiply | `1000.4:mul(2)` → `2000.8` |
| `:div(value)` | v1.2.0+ | Divide | `1000.4:div(2)` → `500.2` |
| `:mod(value)` | v4.5.2+ | Modulo | `3:mod(2)` → `1` |
| `:abs` | v4.12.0+ | Absolute value | `-10:abs` → `10` |
| `:ceil` | v4.22.8+ | Round up to nearest integer | `1.05:ceil` → `2` |
| `:floor` | v4.22.8+ | Round down to nearest integer | `1.05:floor` → `1` |
| `:int` | v1.0.0+ | Convert to integer (UNRECOMMENDED) | — |
| `:toFixed` | v1.0.0+ | Fixed decimal string (UNRECOMMENDED) | — |
| `:toEN` | v1.0.0+ | English number format (UNRECOMMENDED) | — |
| `:toFR` | v1.0.0+ | French number format (UNRECOMMENDED) | — |

**Math formula in formatters** — `add`, `mul`, `sub`, `div` accept simple expressions:
```
{d.val:add(.otherQty + .vat * d.sub.price - 10 / 2)}
```
Only `+`, `*`, `-`, `/` are allowed, without extra parentheses. `*` and `/` have higher precedence.

---

## Currency Formatters

| Formatter | Version | Description |
|---|---|---|
| `:formatC(precisionOrFormat, targetCurrency)` | v1.2.0+ | Format as currency using `lang` + `currency` options |
| `:convCurr(target, source)` | v1.2.0+ | Convert from one currency to another |

**`:formatC` format codes**: `'M'` = currency name in words, `'L'` = symbol + amount, `'LL'` = amount + currency name.

**`:formatC` examples** (with `lang:"en-us"`, `currency.source:"EUR"`, `currency.target:"USD"`, rates `EUR:1, USD:2`):
```
'1000.456':formatC()     // "$2,000.91"
'1000.456':formatC('M')  // "dollars"
'1000':formatC('L')      // "$2,000.00"
'1000':formatC('LL')     // "2,000.00 dollars"
```

**`:convCurr` examples** (same options):
```
10:convCurr()           // 20
1000:convCurr('EUR')    // 1000
1000:convCurr('USD')    // 2000
1000:convCurr('USD', 'USD')  // 1000
```

---

## Date Formatters

Input can be ISO 8601, Unix timestamp (ms), Unix timestamp in seconds (with `'X'` patternIn), or any moment.js-parseable string.

| Formatter | Version | Description |
|---|---|---|
| `:formatD(patternOut, patternIn)` | v3.0.0+ | Format date (patternIn defaults to ISO 8601) |
| `:addD(amount, unit, patternIn)` | v3.0.0+ | Add time to a date |
| `:subD(amount, unit, patternIn)` | v3.0.0+ | Subtract time from a date |
| `:startOfD(unit, patternIn)` | v3.0.0+ | Set to start of unit (day/month/year/etc.) |
| `:endOfD(unit, patternIn)` | v3.0.0+ | Set to end of unit |
| `:diffD(toDate, unit, patternFromDate, patternToDate)` | v3.0.0+ | Difference between two dates |
| `:convDate(patternIn, patternOut)` | — | Convert date format (older formatter) |

**Available units for `:addD`/`:subD`/`:startOfD`/`:endOfD`**: `day`, `week`, `month`, `quarter`, `year`, `hour`, `minute`, `second`, `millisecond`

**Common moment.js format tokens**: `YYYY` `MM` `DD` `HH` `mm` `ss` `L` (short date) `LL` (long date) `LLL` (date+time) `LLLL` (full) `LT` (time) `dddd` (weekday name) `X` (unix seconds)

**Examples** (with `lang:"en-us"`, `timezone:"Europe/Paris"`):
```
'20160131':formatD('L')       // "01/31/2016"
'20160131':formatD('LL')      // "January 31, 2016"
'20160131':formatD('LLLL')    // "Sunday, January 31, 2016 12:00 AM"
1410715640:formatD('LLLL','X') // unix timestamp to date
'20160131':addD('3','day')    // "2016-02-03T00:00:00.000Z"
'20160131':startOfD('month')  // "2016-01-01T00:00:00.000Z"
```

---

## Interval Formatter

| Formatter | Version | Description |
|---|---|---|
| `:formatI(patternOut, patternIn)` | v4.1.0+ | Format intervals/durations. Input defaults to milliseconds. |

**patternOut values**: `human` `human+` `millisecond(s)`/`ms` `second(s)`/`s` `minute(s)`/`m` `hour(s)`/`h` `day(s)`/`d` `week(s)`/`w` `month(s)`/`M` `year(s)`/`y`

**Examples** (with `lang:"en"`):
```
2000:formatI('second')     // 2
3600000:formatI('hour')    // 1
2000:formatI('human')      // "a few seconds"
2000:formatI('human+')     // "in a few seconds"
-2000:formatI('human+')    // "a few seconds ago"
60:formatI('ms','minute')  // 3600000
'P1Y2M3DT4H5M6S':formatI('hour')  // ISO 8601 duration input
```

---

## Array Formatters

| Formatter | Version | Description |
|---|---|---|
| `:arrayJoin(separator, index, count)` | v4.12.0+ | Flatten array to string |
| `:arrayMap(objSeparator, attSeparator, attributes)` | v0.12.5+ | Flatten array of objects to string |
| `:count(start)` | v1.1.0+ | Row counter. `:cumCount` is the modern equivalent (v4+) and additionally supports `partitionBy` |

**`:arrayJoin` examples**:
```
['homer','bart','lisa']:arrayJoin()       // "homer, bart, lisa"
['homer','bart','lisa']:arrayJoin(' | ')  // "homer | bart | lisa"
['homer','bart','lisa']:arrayJoin('',1)   // "bartlisa"  (skip first)
['homer','bart','lisa']:arrayJoin('',1,1) // "bart"      (1 item from index 1)
['homer','bart','lisa']:arrayJoin('',0,-1)// "homerbart" (all but last)
```

**`:arrayMap` examples**:
```
[{id:2,name:'homer'},{id:3,name:'bart'}]:arrayMap()            // "2:homer, 3:bart"
[{id:2,name:'homer'},{id:3,name:'bart'}]:arrayMap(' ; ','|')   // "2|homer ; 3|bart"
[{id:2,name:'homer'},{id:3,name:'bart'}]:arrayMap(' ; ','|','id') // "2 ; 3"
```

---

## Aggregator Formatters (ENTERPRISE, v4+)

Used to compute aggregates over arrays. Can be used standalone (outside loop) or inside a loop with optional `partitionBy` parameter.

| Formatter | Description |
|---|---|
| `:aggSum(partitionBy?)` | Sum of all values |
| `:aggAvg(partitionBy?)` | Average |
| `:aggMin(partitionBy?)` | Minimum value |
| `:aggMax(partitionBy?)` | Maximum value |
| `:aggCount(partitionBy?)` | Count of items |
| `:aggCountD(partitionBy?)` | Count of distinct items |
| `:aggStr(separator, partitionBy?)` | Concatenate all strings |
| `:aggStrD(separator, partitionBy?)` | Concatenate distinct strings |
| `:cumSum(partitionBy?)` | Cumulative running sum |
| `:cumCount(partitionBy?)` | Sequential row counter |
| `:cumCountD(partitionBy?)` | Sequential counter for distinct items |
| `:aggSum:cumSum` | Combined formatter for nested arrays: subtotal per group + running total across groups |

**Standalone example** (outside loop — returns total):
```
{d.cars[].qty:aggSum}               ← sum of all
{d.cars[sort>1].qty:aggSum}         ← sum with array filter
{d.cars[].qty:mul(.sort):aggSum}    ← chain formatters before agg
```

**Loop example** (inside loop — shows value per row):
```
{d[i].brand}   {d[i].qty:aggSum}    {d[i].qty:cumSum}
{d[i+1].brand}
```

**Partitioned example** (sub-total per group):
```
{d[i].brand}   {d[i].qty:aggSum(.brand)}
{d[i+1].brand}
```

**Nested array subtotals**:
```
{d[i].country}   {d[i].cities[].cars:aggSum}
{d[i+1].country}
```

**Sub-object limitation** — use `:print` to navigate into sub-objects first:
```
{d[].items:print(.sub.qty):aggSum}
```

**`:aggSum:cumSum` combined** — valid combined formatter for nested arrays. `:aggSum` computes the subtotal per group, `:cumSum` then tracks the running total across those grouped results:
```
{d[i].cities[].cars:aggSum:cumSum}
{d[i+1]}
```

---

## Condition Formatters

### Logical operators (must be followed by an output formatter)

| Formatter | Meaning |
|---|---|
| `:ifEQ(val)` | equals |
| `:ifNE(val)` | not equal |
| `:ifGT(val)` | greater than |
| `:ifGTE(val)` | greater than or equal |
| `:ifLT(val)` | less than |
| `:ifLTE(val)` | less than or equal |
| `:ifIN(val)` | contains (string or array member) |
| `:ifNIN(val)` | does not contain |
| `:ifEM()` | is empty (null, undefined, [], {}, '') |
| `:ifNEM()` | is not empty |
| `:ifTE(type)` | type equals ('string', 'number', 'boolean', 'array', 'object') |
| `:and(val?)` | AND the next condition |
| `:or(val?)` | OR the next condition (default between conditions) |

### Output formatters (follow a logical operator)

| Formatter | Description |
|---|---|
| `:show(msg)` | Print `msg` if condition is true |
| `:elseShow(msg)` | Print `msg` if condition is false |
| `:showBegin` / `:showEnd` | Show block between tags if condition is true |
| `:hideBegin` / `:hideEnd` | Hide block between tags if condition is true |
| `:drop(element, n?)` | Drop element if condition is true (ENTERPRISE) |
| `:keep(element, n?)` | Keep element if condition is true, drop otherwise (ENTERPRISE) |

**`:drop` / `:keep` elements**: `'row'` `'p'` `'img'` `'table'` `'chart'` `'shape'` `'slide'` `'item'` `'sheet'` `'h'`

Note: no formatters can be chained after `drop`, `keep`, `hideBegin`, `hideEnd`, `showBegin`, `showEnd`.

### Legacy condition formatters (deprecated, use new ones above)

| Formatter | Replacement |
|---|---|
| `:ifEmpty(textIfTrue)` | `:ifEM():show(...)` |
| `:ifEqual(value, textIfTrue)` | `:ifEQ(value):show(...)` |
| `:ifContain(value, textIfTrue)` | `:ifIN(value):show(...)` |

---

## Store and Transform — `:set` (ENTERPRISE, v5+ / v4 with preReleaseFeatureIn)

| Usage | Description |
|---|---|
| `:set(c.varName)` | Store result into complement object |
| `:set(d.varName)` | Store result into data object |
| `:set(.relativePath)` | Add/modify attribute on each item in a looped array |
| `:set(c.newArr[])` | Clone array into a new array |
| `:set(c.arr[field=.value].prop)` | Group/merge arrays with join expression |

Tags using `:set` print **nothing**. Execution order is guaranteed within same document section.

---

## Key-value Mapping

| Formatter | Version | Description |
|---|---|
| `:convEnum('ENUM_NAME')` | v0.13.0+ | Map index or key to label via options enum |

**Example** (with `enum: { ORDER_STATUS: ['pending','sent','delivered'] }`):
```
0:convEnum('ORDER_STATUS')  // "pending"
1:convEnum('ORDER_STATUS')  // "sent"
5:convEnum('ORDER_STATUS')  // 5 (out of range → unchanged)
```

---

## Advanced / Misc Formatters

| Formatter | Version | Description |
|---|---|
| `:formatR` | v5.0.4+ | Format a country/region code (in the data) to a human-readable name in the report's language. Accepts ISO 3166-1 Alpha-3 (`FRA`), Alpha-2 (`FR`), or Numeric (`250`). Example: `{d.countryCode:formatR}` |
| `:imageFit(option)` | v3.2.0+ ENTERPRISE | Resize replacement image to placeholder (DOCX/ODT). `fillWidth` (default), `contain`, `fill` |
| `:barcode(type, options?)` | v3.4.6+ ENTERPRISE | Insert barcode/QR as image. Write in alt-text of placeholder image. `type`: `qrcode`, `code128`, `ean13`, `ean8`, `pdf417`, and 100+ others. `svg:true` for vector output. Options (second arg, `key:value` pairs): `width`, `height`, `scale` (1–10), `includetext`, `textsize`, `textxalign` (left/center/right/justify), `textyalign` (below/center/above), `rotate` (N/R/L/I), `barcolor` (#RRGGBB), `textcolor` (#RRGGBB), `backgroundcolor` (#RRGGBB), `eclevel` (L/M/Q/H — QR only) |
| `:chart` | v4.0+ ENTERPRISE | In HTML `<img src="">`: inject ECharts v5 chart. JSON must contain full ECharts config with `type:"echarts@v5a"` |
| `:color(scope, type)` | v4.17.0+ ENTERPRISE | Apply dynamic hex color to text, cells, rows, or shapes — [see full reference](#color-formatter--full-reference-enterprise-v417) |
| `:appendFile(where?)` | v4.22.0+ ENTERPRISE | Append PDF at end (`'end'`, default) or `'start'` of generated PDF |
| `:attachFile(name, type)` | v4.23.0+ ENTERPRISE | Attach file inside PDF (Factur-X, ZUGFeRD). Auto-detects Factur-X level when filename is `'factur-x.xml'` |
| `:appendTemplate(id, pos?)` | v5.0.3+ ENTERPRISE On-Premise | Generate another template with current data and append its PDF. `pos`: `'end'` (default) or `'start'`. Works in loops |
| `:defaultURL(url)` | v3.1.6+ ENTERPRISE | Fallback URL if dynamic hyperlink is invalid. Place before `:html` when combined |
| `:sign` | v5.0.0+ ENTERPRISE BETA | Place signature field; returns position coordinates in API response. Prints nothing |
| `:transform(axis, unit)` | v4.22.11+ ENTERPRISE | Move shapes on X or Y axis. Requires `{o.preReleaseFeatureIn=4022011}`. axis: `'x'`/`'y'`. unit: `'cm'` `'mm'` `'inch'` `'pt'` `'in'` |
| `:svgUpdate(attrName, selectorVal, selectorType?)` | v5-beta.14+ | Update SVG attribute on matching elements. Tag goes in SVG HTML comment. selectorType: `'id'`, `'id > *'`, `'id *'` (default). Supports: rect, circle, line, ellipse, polygon, polyline, path |
| `:svgSelectiveUpdate(attrName, attrVal, selectorType?)` | v5-beta.14+ | Select SVG elements by current value and inject attribute. Same selectorType options |

---

## `:html` Formatter — Full Reference (ENTERPRISE, v5+)

Compatible with: ODT, DOCX, HTML, PDF.

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

Supported color formats for DOCX/ODT: hex (`#FF00BB`), RGB (`rgb(31,120,50)`), HSL (`hsl(120,75,25)`), named CSS colors (`red`, `lightseagreen`, etc.)

### Options (combinable with comma)

| Option | Effect | Example |
|---|---|---|
| `inline` | Render inside current paragraph. Only: `<a>` `<b>` `<strong>` `<em>` `<i>` `<s>` `<del>` `<u>` | `{d.v:html(inline)}` |
| `nospace` | Remove empty paragraph added after block elements | `{d.v:html(nospace)}` |
| `tabletheme:Name` | Apply named Word table theme (DOCX only) | `{d.v:html(tabletheme:GridTable2-Accent3)}` |
| `headingtheme:prefix` | Use custom heading styles `prefix1`…`prefix6` | `{d.v:html(headingtheme:my-theme-)}` |

Combine: `{d.value:html(nospace,headingtheme:my-theme-)}`

### Font and paragraph behaviour
- HTML content **inherits** font family and size from the template paragraph where the tag is placed
- Text alignment from the template is **not** retained
- By default, `:html` renders content in a **new paragraph**. Use `inline` to stay in the current paragraph
- Tab characters: use `&ensp;` or `&emsp;` — do NOT use `&#9;` (collapsed to space by HTML parsers)

---

## `:color` Formatter — Full Reference (ENTERPRISE, v4.17+)

```
{d.hexColor:color(scope, type)}
{d.val:ifEQ('ok'):show(007700):elseShow(FF0000):color(row, text)}
```

Color must be 6-digit hex, with or without `#`. Invalid values → replaced with `#888888`.

| `scope` | Element |
|---|---|
| `p` (default) | Current paragraph |
| `row` | Current table row |
| `cell` | Current table cell |
| `shape` | Current shape |
| `part` | Inline section within a paragraph (ODT only) |

| `type` | What changes |
|---|---|
| `text` (default) | Text color |
| `highlight` | Text highlight (**not supported in DOCX**) |
| `background` | Cell/row/shape background |
| `border` | Shape border only |

**Limitations:** Cannot use in aliases. Cannot combine with aggregators. For ODP (text, tables, shapes) and ODT (shapes only): a non-default style must already be applied in the template. Complex nested tables with sub-table colors are not fully supported.

## PDF Form Formatters

Used in PDF form templates only.

| Formatter | Description |
|---|---|
| `:fill` | Fill the text field behind the tag annotation (annotation must overlap the field) |
| `:check` | Check the checkbox or radio button behind the tag annotation |
| `:uncheck` | Uncheck the checkbox behind the tag annotation |
| `:fillField('fieldName')` | Fill a text field by name anywhere in the document. Also checks a checkbox with any non-empty value |
| `:checkField('fieldName')` | Check a checkbox by name if condition is true |

Example — select a radio button option:
```
{d.genre:ifEQ('boy'):show(male):ifEQ('girl'):show(female):fillField('genderGroup')}
```
