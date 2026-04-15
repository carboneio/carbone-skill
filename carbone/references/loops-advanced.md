# Carbone Loops — Advanced Reference

Advanced loop patterns. Read this file when the user asks about: filtering with negative index or OR logic, bidirectional loops, sorting, distinct values, lookup (JOIN), parallel loops, loops on primitive arrays or nested arrays, object attribute search, or row repetition.

---

## Filtering — advanced patterns

**Negative index — access or exclude last N items:**
```
{d[i=-1].name}           ← last item only
{d[i, i!=-1].name}       ← exclude the last item
{d[i, i<-2].name}        ← exclude the last 2 items
{d[i+1, i<-2].name}
```

**Advanced filter with `:set` (v5+)** — for OR logic or computed conditions, compute a flag first then filter:
```
{d[].name:ifIN('Model 3'):or:ifIN('Falcon 9'):show(1):elseShow(0):set(.isShown)}
{d[i].name}
{d[i+1, isShown=1].name}
```

---

## Bidirectional loop (DOCX/HTML/MD — v4.8.0+)
Creates rows AND columns simultaneously:
```
{d.titles[i].name}     {d.titles[i+1].name}
{d.cars[i].models[i]}  {d.cars[i].models[i+1]}
{d.cars[i+1].models[i]}
```

---

## Accessing the loop iterator value
Use `:add(.i)` style to get the current index. Dots indicate level in the hierarchy:
```
{d[i].cars[i].other.wheels[i].tire.subObject:add(.i):add(..i):add(...i)}
```
- `.i` → index of `cars[i]`
- `..i` → index of `d[i]`
- `...i` → index of `wheels[i]`

⚠️ Note: the dot count is currently **inverted** (a known Carbone bug maintained for backward compat).

---

## Sorting
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

---

## Distinct
Print only the first occurrence of each distinct value:
```
{d[type].brand}
{d[type+1].brand}
```

---

## Lookup (v5.2+ — requires pre-release flag)
Join two arrays by matching IDs (like SQL JOIN or VLOOKUP).
Requires `{o.preReleaseFeatureIn=5002000}` in the template.
```
{d.movies[i].actorId:print(..actors[id=.actorId].firstName)}
```

---

## Parallel loop
Access a second array at the same index using the iterator:
```
{d.cars[i].id}   {d.cars[i].id:print(..brands[.i].name)}
{d.cars[i+1].id}
```

---

## Loop on array of strings or numbers (v4.9.0+)
When the array contains primitive values (strings, numbers) rather than objects:
```
{d.myArray[i]}
{d.myArray[i+1]}
```
Direct access also works: `{d.myArray[0]:upperCase}`

⚠️ If at least one tag accesses a property (e.g. `{d.myArray[i].id}`), Carbone treats the array as an array of objects and `{d.myArray[i]}` prints nothing.

---

## Loop on array of arrays (v4.9.0+)
Nested arrays can themselves be arrays (unlimited depth):
```
{d.myArray[i][i].val}
{d.myArray[i][i+1].val}
{d.myArray[i+1].val}
```

---

## Object attribute search (v4.22.5+)
Search an object's properties by attribute name or value (without looping):
```
{d.myObject[.att = jack].val}    ← find value where key = "jack"
{d.myObject[.val = 20].att}      ← find key where value = 20
```
Where `myObject` is `{ paul: '10', jack: '20', bob: '30' }` — `.att` is the key, `.val` is the value.

---

## Row repetition (experimental)
Repeat a row N times based on a JSON value. `qty` is the number of repetitions:
```
{d[i].id} - {d[i+1*qty].id}
```
Maximum: 400 repetitions. Row is duplicated `qty` times; rows with `qty=0` are skipped.
