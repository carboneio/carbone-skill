---
name: carbone
description: >
  Expert assistant for the Carbone Universal Templating Language. Use this skill whenever the user:
  - Asks how to write a Carbone tag, placeholder, loop, condition, or formatter
  - Wants to build or design a Carbone template (invoice, report, contract, spreadsheet, presentation, etc.)
  - Asks you to validate, fix, or explain existing Carbone tag syntax
  - Mentions Carbone, {d.}, {c.}, formatters, merge fields, or document generation from JSON
  - Uses phrases like "Carbone tag", "Carbone template", "how do I loop in Carbone", "Carbone formatter", "aggSum", "hideBegin", "drop row", ":chart", "carbone-pdf-options", "Markdown to PDF", "HTML to PDF"
  - Asks about HTML templates (CSS injection, charts, headers/footers, PDF options) or Markdown templates (loops in tables, convert to DOCX/PDF)
  - Is generating a DOCX, XLSX, or PPTX file and needs to embed Carbone tags into it
  Always consult this skill before answering any question about Carbone syntax or template design.
  NEVER invent syntax — only use what is documented here or in the reference files.
license: Apache-2.0
compatibility: Compatible with any Agent Skills-compatible platform (Claude, Claude Code, ChatGPT, Cursor, VS Code, Gemini CLI, and more). No special environment requirements.
metadata:
  author: carboneio
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
- `{o.useHighPrecisionArithmetic=true}` — enables arbitrary-precision decimal arithmetic (v4.22.4+)
- `{o.preReleaseFeatureIn=VERSION}` — activates a pre-release feature (e.g. `{o.preReleaseFeatureIn=5002000}`)
- `{o.timezone=Europe/Paris}` — force the timezone used by date formatters, overrides the API option (v5.4.3+)
- `{o.lang=en-US}` — force the language/locale, overrides the API option. Accepts uppercase or lowercase (v5.4.3+)
- `{o.converter=L}` — force the document converter: `L` = LibreOffice, `O` = OnlyOffice, `C` = Chrome (v5.4.3+)
- `{o.exportFormattedValuesAsText=true}` — force `:formatN` to output numbers as localized text strings instead of native XLSX number format. **XLSX templates only** (v5.4.3+)
- `{o.preReleaseFeatureIn=5004002}` — enables fix for direct object access combined with object iteration on the same object (v5.4.2, opt-in only)

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

### 4d. Bidirectional loop (DOCX/HTML/MD — v4.8.0+)
Creates rows AND columns simultaneously:
```
{d.titles[i].name}     {d.titles[i+1].name}
{d.cars[i].models[i]}  {d.cars[i].models[i+1]}
{d.cars[i+1].models[i]}
```

### 4e. Accessing the loop iterator value
Use `:add(.i)` style to get the current index. Dots indicate level in the hierarchy:
```
{d[i].cars[i].other.wheels[i].tire.subObject:add(.i):add(..i):add(...i)}
```
- `.i` → index of `cars[i]`
- `..i` → index of `d[i]`
- `...i` → index of `wheels[i]`

⚠️ Note: the dot count is currently **inverted** (a known Carbone bug maintained for backward compat).

### 4f. Sorting
Sort ascending by placing the attribute name as the iterator. Pairs of attribute/iterator define sort priority:
```
{d.cars[power  , i].brand}
{d.cars[power+1, i+1].brand}
```
Multiple sort attributes:
```
{d.cars[power  , sub.size  , i].brand}
{d.cars[power+1, sub.size+1, i+1].brand}
```
⚠️ Descending sort is coming in v5 — not yet available.

### 4g. Filtering
Filter operators: `>`, `<`, `>=`, `<=`, `=`, `!=`
```
{d[i, age > 19, age < 30].name}
{d[i+1, age > 19, age < 30].name}
```
**v5+**: only the `[i+1]` row needs the filter:
```
{d[i].name} {d[i].age}
{d[i+1, age > 19, age < 30].name}
```
Filter first N items: `{d[i, i < 2].name}` / `{d[i+1, i < 2].name}`

**Negative index — access or exclude last N items:**
```
{d[i=-1].name}           ← last item only
{d[i, i!=-1].name}       ← exclude the last item
{d[i, i<-1].name}        ← exclude the last item (equivalent)
{d[i, i<-2].name}        ← exclude the last 2 items
{d[i+1, i<-2].name}
```

**Advanced filter with `:set` (v5+)** — for OR logic or computed conditions,
compute a boolean flag first, then filter on it:
```
{d[].name:ifIN('Model 3'):or:ifIN('Falcon 9'):show(1):elseShow(0):set(.isShown)}
{d[i].name}
{d[i+1, isShown=1].name}
```

### 4h. Grouping (v5+)
Use an attribute name as a custom iterator to group rows by that attribute:
```
{d[brand].brand}   {d[brand].qty:aggSum(.brand)}
{d[brand+1].brand}
```

### 4i. Distinct
Print only the first occurrence of each distinct value:
```
{d[type].brand}
{d[type+1].brand}
```

### 4j. Lookup (v5.2+ — requires pre-release flag)
Join two arrays by matching IDs (like SQL JOIN or VLOOKUP).
Requires `{o.preReleaseFeatureIn=5002000}` in the template.
```
{d.movies[i].actorId:print(..actors[id=.actorId].firstName)}
```

### 4k. Parallel loop
Access a second array at the same index using the iterator:
```
{d.cars[i].id}   {d.cars[i].id:print(..brands[.i].name)}
{d.cars[i+1].id}
```

### 4l. Loop on array of strings or numbers (v4.9.0+)
When the array contains primitive values (strings, numbers) rather than objects:
```
{d.myArray[i]}
{d.myArray[i+1]}
```
Direct access also works: `{d.myArray[0]:upperCase}`

⚠️ If at least one tag accesses a property (e.g. `{d.myArray[i].id}`), Carbone treats the array as an array of objects and `{d.myArray[i]}` prints nothing.

### 4m. Loop on array of arrays (v4.9.0+)
Nested arrays can themselves be arrays (unlimited depth):
```
{d.myArray[i][i].val}
{d.myArray[i][i+1].val}
{d.myArray[i+1].val}
```

### 4n. Object attribute search (v4.22.5+)
Search an object's properties by attribute name or value (without looping):
```
{d.myObject[.att = jack].val}    ← find value where key = "jack"
{d.myObject[.val = 20].att}      ← find key where value = 20
```
Where `myObject` is `{ paul: '10', jack: '20', bob: '30' }` — `.att` is the key, `.val` is the value.

### 4o. Row repetition (experimental)
Repeat a row N times based on a JSON value. `qty` is the number of repetitions:
```
{d[i].id} - {d[i+1*qty].id}
```
Maximum: 400 repetitions. Row is duplicated `qty` times; rows with `qty=0` are skipped.

---

## 5. Formatters

Formatters are chained with `:`. Each formatter's output feeds the next.
```
{d.name:lowerCase:ucFirst}
{d.birthday:formatD(LL)}
```
Parameters use parentheses. Strings with spaces or commas need single quotes: `:prepend('hello world')`.

**Escaping a single quote inside a parameter**: use two adjacent single quotes — `'David''s Car'` (not a double quote).

**Translation tag as a formatter argument** — `{t()}` can be placed **anywhere**, including inside formatter parameters:
```
{d.id:ifEQ(2):show( {t(monday)} ):elseShow( {t(tuesday)} )}
```

**Whitespace is preserved without quotes** — unlike formatter string arguments, `{t()}` does not need single quotes around its content:
```
{t(all whitespace is preserved without quotes)}   ✅
{t(meeting)}                                      ✅
```

**Applying formatters to an injected translation** — chain formatters after `:show` to transform the translated string:
```
{d:show('{t(Destination)}'):ucFirst}    ← capitalises first letter of the translated value
```

**Whitespace in JSON key names** (requires `{o.preReleaseFeatureIn=4022011}`):

Wrap the key name in single quotes to access it:
```
{d.'my param'.'second level'[1].'sub obj'}
{d.byPhase[0].drugs[0].events.'Q3 2025'}
```
⚠️ Not supported inside formatters, array filters, or aliases.

**Dynamic parameters** start with `.` (relative path) or `d.`/`c.` (absolute path):
```
{d.subObject.qtyB:add(.qtyC)}        ← relative path
{d.subObject.qtyB:add(d.qtyA)}       ← absolute path
```

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

| Element | What is dropped |
|---|---|
| `row` | Table row |
| `p` | Paragraph |
| `img` | Image |
| `table` | Table |
| `chart` | Chart |
| `shape` | Shape |
| `slide` | Slide (ODP only) |
| `item` | List item (ODP/ODT only) |
| `sheet` | Sheet (ODS only) |
| `h` | Heading/custom style (ODT only) |

Drop current + next N rows/paragraphs: `{d.text:ifEM():drop(row, 3)}`
Use `:keep` to do the opposite (keep element if condition is true, drop otherwise).

Format support notes: ODS only supports `row` and `img`. HTML only supports `table`, `row`, `p`. XLSX only supports `row`. PPTX supports `row`, `img`, `p`, `shape`, `chart`, `table`.

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

Store a computed value into `c.`:
```
{d.cars[].qty:aggSum:set(c.mySum)}
Total: {c.mySum}
```

Add a new attribute to each item in an array (relative path `.`):
```
{d.cars[].qty:append(' tyres'):set(.newInfo)}
```

Clone / transform an array:
```
{d.myArray[]:set(c.new[])}
{d.myArray[].country:set(c.newArr[].country)}
```

Group flat DB data into nested arrays (distinct merge):
```
{d[].city:set(c.countries[id=.country].cities[].name)}
```

**Transform an object into a searchable flat list** — useful when object keys are dynamic/unknown:

When `sanger_data` is an object with dynamic keys (e.g. a sample ID like `VIA097341` that changes every time), you cannot reference it directly. Use `:set` to convert the object into a flat array where `.att` = key and `.val` = value, then filter by known keys to isolate the unknown one:

```
{d.sanger_data[].val:set(..list[attr=.att].val)}
```

This creates a `list` array at the same level as `sanger_data`, where each item has `attr` (the original key) and `val` (the original value). You can then use `!=` filters to exclude all known keys — whatever row remains is your dynamic key:

```
{d.sanger_data.sanger_conf:ifNE('Yes'):drop(p)}
Mother – {d.sanger_data.mother}
Father – {d.sanger_data.father}
{d.list[i, attr!='mother', attr!='father', attr!='sanger_conf', attr!='proband_zygosity'].attr} – {d.list[i, attr!='mother', attr!='father', attr!='sanger_conf', attr!='proband_zygosity'].val}
{d.list[i+1, attr!='mother', attr!='father', attr!='sanger_conf', attr!='proband_zygosity'].attr}
```

Note: this uses `..list` (relative path — one level up from `sanger_data`) rather than `c.list`. The `!=` operator is valid in array filters alongside `>`, `<`, `>=`, `<=`, `=`.

**Rules**:
- Tags using `:set` print **nothing**
- Use `c.` for new computed data
- Only alphanumeric characters allowed for newly created keys
- `[i]` iterator cannot be used with `:set`

**Building a dynamic URL with a static base path** — store a constant string into `c.` first, then prepend it onto a dynamic value:
```
{d:print('https://www.website.org/cgi-bin/carddisp.pl?gene='):set(c.url)}
{d.in_report_variants.gene_name:prepend(c.url)}
```
Result: `https://www.website.org/cgi-bin/carddisp.pl?gene=BRCA1` (where `gene_name = "BRCA1"`).
This avoids repeating the full base URL on every tag and makes it easy to update in one place.

**Create a new array from a static comma-separated string defined inline:**
```
{d:print('val1,val2,val3'):split(','):set(d.values)}
```
Useful when you need a fixed list inside the template without adding it to the JSON.

**Get the newest date from a filtered list:**
Convert each date to a Unix timestamp first (ISO date strings cannot be compared with `aggMax` beyond year-level — months and days require numeric comparison), store the timestamp, aggregate the max, flag the winner, then filter:
```
{d.items[status!=''].endDate:formatD(X):set(.endDateX)}
{d.items[status!=''].endDateX:aggMax:ifEQ(.endDateX):show('true'):elseShow('false'):set(.newest)}
{d.items[newest=true].endDate}
```
⚠️ Do NOT use `aggMax` directly on date strings — it compares lexicographically and gives correct results only for year-level comparisons, not months or days.

**Store multiple aggregation results then combine in a complex condition:**
When conditions span multiple aggregations, store each result first, then evaluate:
```
{d.tableA.contacts[].id:aggCount:set(c.totalA)}
{d.tableB.contacts[].id:aggCount:set(c.totalB)}
{c.totalA:ifEQ(0):and(c.totalB):ifEQ(0):show('<br/>'):elseShow('')}
```

**Filter a parent list based on child list content:**
Use `:aggStr` + `:ifIN` + `:set` to write a boolean flag onto each parent based on whether a target value exists in its children:
```
{d.items[].subItems[].label:aggStr('',..name):ifIN('Target Value'):show(true):elseShow(false):set(..hasTarget)}
```
Then filter the parent loop normally:
```
{d.items[hasTarget=true, i].name}
{d.items[hasTarget=true, i+1].name}
```

**Type conversion with `:set` — convert a number stored as a string into a real integer:**
```
{d.items[].data.amount:int:set(.amountInt)}
```
⚠️ `:int` only works for integers. Carbone cannot convert a float stored as a string into a real float type. Note `:int` is deprecated but remains the only way to perform this type coercion with `:set`.

**Retrieve the last item from a child list using iterator index:**
Use `aggMax` on `.i` to find the index of the last child item, store it, then use as a filter:
```
{d.items[].children[].id:aggMax(.i):set(..lastChildIdx)}
{d.items[i].children[id=..lastChildIdx].label}
{d.items[i+1]}
```

**Paginate a flat list into N-column rows:**
Assign a row number to each item using `cumCount`, then group by row into a new structure:
```
{d.items[].name:cumCount:div(3):ceil:set(.row)}
{d.items[].row:set(c.grid[row=.row].row)}
{d.items[]:set(c.grid[row=.row].cells[])}
```
Replace `3` with your column count. Each `c.grid[i]` will have a `cells[]` array of up to 3 items.

**Multi-level JSON restructuring — pair two sibling arrays into a combined child list:**
Use `cumCount(.i)` to generate a unique ID per item within the same parent, then use that ID as the join key to align the two sibling arrays:
```
{d.source.items[].name:set(c.output[key=.key].key)}
{d.source.items[].children[]:set(c.output[key=...key].childItems[])}
{d.source.items[].sideA[].value:cumCount(.i):set(.id)}
{d.source.items[].sideA[].id:set(c.output[key=...key].pairs[id=.id].id)}
{d.source.items[].sideA[].value:set(c.output[key=...key].pairs[id=.id].before)}
{d.source.items[].sideB[].value:set(c.output[key=...key].pairs[id=.id].after)}
```

**Group a flat list by a computed attribute:**
Extract the grouping key into a local variable first, then build the grouped structure and compute totals:
```
{d.records[].price.qty:set(..p)}
{d.records[].p:set(c.groups[price=.p].price)}
{d.records[].price.volume:set(c.groups[price=..p].volume)}
{d.records[].label:set(c.groups[price=.p].label)}
{d.records[]:print(.qty.qty):aggSum(.p):set(c.groups[price=.p].total)}
{c.groups[].total:mul(.price):set(.grandTotal)}
```

**Restructure two flat lists into a grouped structure with row pagination:**
Merge two lists by a shared key, then paginate items into rows of N columns:
```
{d.categories[]:set(c.output[name=.name])}
{d.items[]:set(c.output[name=.category].items[])}
{c.output[].items[].name:cumCount(..name):div(2):ceil:set(.row)}
{c.output[].items[].row:set(..rows[row=.row].row)}
{c.output[].items[]:set(c.output[name=..name].rows[row=.row].items[])}
```
Replace `2` with your column count.

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

### 8d. HTML content rendering (ENTERPRISE, v5+)
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

### 8e. Hyperlinks (ENTERPRISE, v3+)
Set the URL of a hyperlink (in your editor) to a Carbone tag, e.g. `{d.url}`. Works in DOCX, XLSX, PPTX, ODT, ODS, ODP, ODG — on text, images, tables, and lists.

**Important notes:**
- If no protocol is provided, Carbone automatically prepends `https://`
- You cannot mix a static URL with a dynamic tag in the same link — `https://carbone.io{d.path}` won't work; Carbone keeps only the dynamic part
- **XLSX only**: do NOT wrap the tag in `{}`. Use `d.url` (not `{d.url}`) as the hyperlink URL. If `http://` appears before it, it still works.

**`:defaultURL(url)`** — if the injected URL fails validation, use this fallback:
```
{d.documentUrl:defaultURL('https://carbone.io')}
{d.url:defaultURL(.urlOnError)}          ← dynamic fallback
{d.content:defaultURL(.urlOnError):html} ← place before :html
```

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

**`:appendTemplate(templateIdOrVersionId, position?)`** (v5.0.3+, On-Premise and Carbone Cloud as of v5.4.3) — generate another stored template with the current data and append its PDF output:

```
{d.myArray[i]:appendTemplate(110212)}
{d.myArray[i+1]}
```

- `position`: where the document is inserted — `'start'` (beginning) or `'end'` (default)
- **Data forwarding**: each iterated array item becomes the root `{d.}` of the nested template. So `{d.myArray[i]}` is received as `{d.}` inside the appended template
- **Options forwarding**: `lang`, `currency`, `enum`, `converter`, `translations`, `complement` from the original request are all automatically forwarded to the nested template
- The tag prints nothing. Ignored if value is null, undefined, empty string, or final output is not PDF
- ⚠️ No protection against infinite loops (a template appending itself)

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

### 8n. Dynamic Forms — ODT Text Fields and Checkboxes (ENTERPRISE, v3.3+)

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

### 8o. Native Charts (ENTERPRISE, v4+)

**DOCX/Word** — insert a chart via `Insert > Chart`, edit the embedded Excel spreadsheet with Carbone loop tags directly in cell values:
```
{d.temps[i].date}   {d.temps[i].min}   {d.temps[i].max}
{d.temps[i+1].date}
```
⚠️ For native DOCX charts, each `[i]` expression must have its corresponding `[i+1]` on the next row — you cannot use a single `[i+1]` for multiple `[i]` tags.

**ODT/LibreOffice** — insert a chart, edit the Data Table. Use `{bindChart(refValue) = d.tag}` in the Categories column to bind a JSON value to a chart reference value:
```
{d.cheeses[i].type} {bindChart(3)=d.cheeses[i].purchasedTonnes}
{d.cheeses[i+1].type} {bindChart(4)=d.cheeses[i+1].purchasedTonnes}
```
`bindChart(3)` means: replace the value `3` in the chart with the result of the tag.

**ECharts (all formats)** — insert a placeholder image, write `:chart` tag in its alt text:
```
{d.chartOptions:chart}
```
JSON must contain a full ECharts v5 config:
```json
{ "type": "echarts@v5a", "width": 600, "height": 400, "theme": null, "option": { /* ECharts config */ } }
```
Compatible with: ODT, ODS, ODP, ODG, DOCX, PPTX, XLSX, HTML, PDF. Use `echarts@v5a` (not `echarts@v5`) for better SVG compatibility.

---

## 9. Design Best Practices

### Editor configuration (critical!)
Text editors auto-replace straight quotes `'` with "smart quotes" — **Carbone only accepts straight single quotes** in formatter parameters.

- **MS Word**: `Tools > AutoCorrect Options > AutoFormat As You Type` → uncheck "Straight quotes with smart quotes"
- **LibreOffice**: `Tools > AutoCorrect > Localized Options` → disable Replace for single and double quotes

### Tag styling
Only the font style and size of the **first `{` character** applies to the output. You can make the rest of the tag tiny (e.g. size 1pt) to reduce visual clutter in the template.

### Template type choice
- **PDF output**: prefer DOCX or ODT — better pagination, headers/footers, text overflow handling
- **Presentations**: PPTX/ODP use absolute positioning — if a loop generates too much content for one slide, a new slide is NOT created automatically. Use loop filters to paginate manually
- **Spreadsheets**: XLSX/ODS formulas survive, but Carbone injects data first and formulas recalculate after. Formulas must account for row shifts from loops

### Numbers in spreadsheets (XLSX/ODS)
Use `:formatN` to ensure Carbone tags output native Excel numbers (not strings):
```
Cell A1: {d.val:formatN}
Cell B1: =A1*100    ← now works as a formula
```

### XLSX/ODS — Mixing formulas with Carbone tags is NOT supported

It is **not possible** to embed Excel formula syntax inside a Carbone tag. The following is invalid and will not work:
```
{d.term1.eventDates[" & COLUMN()-6 & "].displayDate}   ← ❌ NOT supported
```
Carbone tags and Excel formulas must be kept in **separate cells**. A Carbone tag is replaced with a value before Excel formulas run — the two systems cannot be interleaved.

### XLSX/ODS — Computing totals when Carbone injects new rows

This is a common challenge: a `SUM(B4:B5)` formula breaks when Carbone pushes rows down dynamically. Two solutions:

**Solution 1 — Dynamic Excel formula using `INDIRECT` + `ROW`**

Place this in the total cell, below the loop. It sums everything in column B from row 4 up to the row just above:
```
=SUM(INDIRECT("B4:B" & ROW()-1))
```

For bi-directional tables, or when you want the formula to work in **any cell** in the sheet:
```
=SUM(INDIRECT(ADDRESS(1,COLUMN()) & ":" & ADDRESS(ROW()-1,COLUMN())))
```
This sums all numbers in the same column above the current cell, regardless of which row or column it's placed in.

**Solution 2 — Carbone aggregator (recommended)**

Let Carbone compute the total before Excel even opens the file:
```
{d.action[].time:aggSum:formatN}
```
⚠️ Always add `:formatN` at the end — without it, the result is a string and Excel formulas won't be able to use it in calculations.

**Extra tip — reference the total elsewhere using `MATCH`/`INDEX`**

If you need to reference the total in another formula, add a static label next to it (e.g. `TOTAL weeks` in column A), then use:
```
=INDEX(B:B, MATCH("TOTAL weeks", A:A, 0))
```
This finds the total row dynamically regardless of how many rows Carbone injected.

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
**Do not** resize table columns by mouse-dragging the borders — it creates imprecise widths that can cause layout issues. Instead: select the column → right-click → **Table Properties → Column → set "Preferred width"** to an exact value.
SVG files can be used as templates. Update SVG element attributes dynamically using two formatters. Write the tags inside HTML comments in the SVG (`<!-- {d.value:svgUpdate(...)} -->`).

**`:svgUpdate(attrName, selectorValue, selectorType)`** — update an SVG attribute on elements matching a selector:
```
{d.myTable[].width:svgUpdate('width', .id)}
```
- `attrName`: SVG attribute to update (e.g. `'fill'`, `'stroke'`, `'width'`)
- `selectorValue`: value used to find the target element (matches against the `selectorType` attribute)
- `selectorType`: how to select — `'id'` (exact match), `'id > *'` (direct children), `'id *'` (all descendants, default)

**`:svgSelectiveUpdate(attrName, attrValue, selectorType)`** — select elements and inject a value:
```
{d.myTable[active=true].id:svgSelectiveUpdate('fill', .color)}
```
Only basic SVG shapes are updated: `rect`, `circle`, `line`, `ellipse`, `polygon`, `polyline`, `path`.

### 8m. PDF form filling (ENTERPRISE, v5.0.0-beta.11+)
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
- `references/html-templates.md` — HTML-specific features: inline CSS injection, `:chart`, barcodes in `<img src>`, `<carbone-pdf-header/footer>`, page breaks, `<carbone-pdf-options>`, PDF conversion API
- `references/markdown-templates.md` — Markdown-specific features: table loops, limitations (no barcodes, no page breaks, no HTML injection yet), convert to PDF/DOCX/ODT/PNG/JPG

**When to read which reference:**
- User asks about HTML template, CSS injection, chart formatter, `<carbone-pdf-options>`, headers/footers → read `references/html-templates.md`
- User asks about Markdown template, MD loops, converting MD to PDF/DOCX → read `references/markdown-templates.md`

Official docs: https://carbone.io/documentation/design/overview/getting-started.html
