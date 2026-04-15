# Carbone `:set` — Advanced Patterns Reference

Read this file when the user needs to solve complex data transformation problems using `:set`: building dynamic URLs, type conversion, date aggregation, JSON restructuring, flat list grouping, pagination into N-column rows, pairing sibling arrays, filtering a parent list based on child content, or creating arrays from static strings.

---

## Building a dynamic URL with a static base path
Store a constant string into `c.` first, then prepend it onto a dynamic value:
```
{d:print('https://example.com/api/details?ref='):set(c.url)}
{d.report_items.item_code:prepend(c.url)}
```
Result: `https://example.com/api/details?ref=ITEM001` (where `item_code = "ITEM001"`).
This avoids repeating the full base URL on every tag and makes it easy to update in one place.

---

## Create a new array from a static comma-separated string defined inline
```
{d:print('val1,val2,val3'):split(','):set(d.values)}
```
Useful when you need a fixed list inside the template without adding it to the JSON.

---

## Get the newest date from a filtered list
Convert each date to a Unix timestamp first (ISO date strings cannot be compared with `aggMax` beyond year-level — months and days require numeric comparison), store the timestamp, aggregate the max, flag the winner, then filter:
```
{d.items[status!=''].endDate:formatD(X):set(.endDateX)}
{d.items[status!=''].endDateX:aggMax:ifEQ(.endDateX):show('true'):elseShow('false'):set(.newest)}
{d.items[newest=true].endDate}
```
⚠️ Do NOT use `aggMax` directly on date strings — it compares lexicographically and gives correct results only for year-level comparisons, not months or days.

---

## Store multiple aggregation results then combine in a complex condition
When conditions span multiple aggregations, store each result first, then evaluate:
```
{d.tableA.contacts[].id:aggCount:set(c.totalA)}
{d.tableB.contacts[].id:aggCount:set(c.totalB)}
{c.totalA:ifEQ(0):and(c.totalB):ifEQ(0):show('<br/>'):elseShow('')}
```

---

## Filter a parent list based on child list content
Use `:aggStr` + `:ifIN` + `:set` to write a boolean flag onto each parent based on whether a target value exists in its children:
```
{d.items[].subItems[].label:aggStr('',..name):ifIN('Target Value'):show(true):elseShow(false):set(..hasTarget)}
```
Then filter the parent loop normally:
```
{d.items[hasTarget=true, i].name}
{d.items[hasTarget=true, i+1].name}
```

---

## Type conversion — convert a number stored as a string into a real integer
```
{d.items[].data.amount:int:set(.amountInt)}
```
⚠️ `:int` only works for integers. Carbone cannot convert a float stored as a string into a real float type. Note `:int` is deprecated but remains the only way to perform this type coercion with `:set`.

---

## Retrieve the last item from a child list using iterator index
Use `aggMax` on `.i` to find the index of the last child item, store it, then use as a filter:
```
{d.items[].children[].id:aggMax(.i):set(..lastChildIdx)}
{d.items[i].children[id=..lastChildIdx].label}
{d.items[i+1]}
```

---

## Paginate a flat list into N-column rows
Assign a row number to each item using `cumCount`, then group by row into a new structure:
```
{d.items[].name:cumCount:div(3):ceil:set(.row)}
{d.items[].row:set(c.grid[row=.row].row)}
{d.items[]:set(c.grid[row=.row].cells[])}
```
Replace `3` with your column count. Each `c.grid[i]` will have a `cells[]` array of up to 3 items.

---

## Multi-level JSON restructuring — pair two sibling arrays into a combined child list
Use `cumCount(.i)` to generate a unique ID per item within the same parent, then use that ID as the join key to align the two sibling arrays:
```
{d.source.items[].name:set(c.output[key=.key].key)}
{d.source.items[].children[]:set(c.output[key=...key].childItems[])}
{d.source.items[].sideA[].value:cumCount(.i):set(.id)}
{d.source.items[].sideA[].id:set(c.output[key=...key].pairs[id=.id].id)}
{d.source.items[].sideA[].value:set(c.output[key=...key].pairs[id=.id].before)}
{d.source.items[].sideB[].value:set(c.output[key=...key].pairs[id=.id].after)}
```

---

## Group a flat list by a computed attribute
Extract the grouping key into a local variable first, then build the grouped structure and compute totals:
```
{d.records[].price.qty:set(..p)}
{d.records[].p:set(c.groups[price=.p].price)}
{d.records[].price.volume:set(c.groups[price=..p].volume)}
{d.records[].label:set(c.groups[price=.p].label)}
{d.records[]:print(.qty.qty):aggSum(.p):set(c.groups[price=.p].total)}
{c.groups[].total:mul(.price):set(.grandTotal)}
```

---

## Transform an object with dynamic keys into a searchable flat list
Useful when object keys are dynamic/unknown (e.g. a product SKU like `SKU20240` that changes every time). Use `:set` to convert the object into a flat array where `.att` = key and `.val` = value, then filter by known keys to isolate the unknown one:
```
{d.product_data[].val:set(..list[attr=.att].val)}
```
This creates a `list` array at the same level as `product_data`. Then use `!=` filters to exclude known keys:
```
{d.product_data.is_certified:ifNE('Yes'):drop(p)}
Supplier – {d.product_data.supplier}
Origin – {d.product_data.origin}
{d.list[i, attr!='supplier', attr!='origin', attr!='is_certified', attr!='item_grade'].attr} – {d.list[i, attr!='supplier', attr!='origin', attr!='is_certified', attr!='item_grade'].val}
{d.list[i+1, attr!='supplier', attr!='origin', attr!='is_certified', attr!='item_grade'].attr}
```
Note: uses `..list` (relative path — one level up from `product_data`). The `!=` operator is valid in array filters alongside `>`, `<`, `>=`, `<=`, `=`.

---

## Restructure two flat lists into a grouped structure with row pagination
Merge two lists by a shared key, then paginate items into rows of N columns:
```
{d.categories[]:set(c.output[name=.name])}
{d.items[]:set(c.output[name=.category].items[])}
{c.output[].items[].name:cumCount(..name):div(2):ceil:set(.row)}
{c.output[].items[].row:set(..rows[row=.row].row)}
{c.output[].items[]:set(c.output[name=..name].rows[row=.row].items[])}
```
Replace `2` with your column count.
