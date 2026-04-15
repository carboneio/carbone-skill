# Carbone XLSX/ODS — Formula Tips

Read this file when the user asks about dynamic Excel formulas that survive Carbone row injection, computing totals after a loop, or referencing injected rows with MATCH/INDEX.

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
