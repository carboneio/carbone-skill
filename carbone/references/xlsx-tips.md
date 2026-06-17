# Carbone XLSX/ODS — Formula Tips

Read this file when the user asks about dynamic Excel formulas that survive Carbone row injection, computing totals after a loop, referencing injected rows with MATCH/INDEX, or why a cell's percentage/currency/date format is not being applied.

---

## Number formats only apply to numeric cells

In XLSX/ODS, the cell's Excel number format (percentage, currency, thousands, date) is applied by the spreadsheet — but **only to native numeric cells**. If Carbone writes the value as text, the format is silently ignored and the raw value is shown verbatim.

A value reaches the cell as text when the JSON field is a string (`"0.032"`), or when `{o.exportFormattedValuesAsText=true}` is set. In XLSX/ODS templates, `:formatN` is the **only** way to force Carbone to write a native number into the cell — arithmetic formatters like `:mul(1)` or `:add(0)` do not change the output type. Once the cell holds a real number, the cell's own format handles display:
```
{d.rate:formatN}    ← cell formatted as 0.00%       → 3.20%
{d.price:formatN}   ← cell formatted as $#,##0.00   → $1,234.50
```

**The `:formatN` precision argument is ignored in XLSX/ODS.** Because the cell's own number format controls decimal display, `:formatN`, `:formatN(0)`, and `:formatN(2)` all behave identically — Carbone writes the raw native number and lets Excel/Calc format it. If you need a specific number of decimals, set them on the cell format, not on the tag.

---

## Dynamic Excel formula using `INDIRECT` + `ROW`

A `SUM(B4:B5)` formula breaks when Carbone pushes rows down dynamically. Place this in the total cell below the loop — it sums everything in column B from row 4 up to the row just above:
```
=SUM(INDIRECT("B4:B" & ROW()-1))
```

For bi-directional tables, or when you want the formula to work in **any cell** in the sheet:
```
=SUM(INDIRECT(ADDRESS(1,COLUMN()) & ":" & ADDRESS(ROW()-1,COLUMN())))
```
Sums all numbers in the same column above the current cell, regardless of row or column position.

## Reference the total elsewhere using `MATCH`/`INDEX`

Add a static label next to the total cell (e.g. `TOTAL weeks` in column A), then reference it dynamically:
```
=INDEX(B:B, MATCH("TOTAL weeks", A:A, 0))
```
Finds the total row dynamically regardless of how many rows Carbone injected.
