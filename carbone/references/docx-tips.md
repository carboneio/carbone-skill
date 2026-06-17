# Carbone DOCX/ODT — Section & Layout Tips

Read this file when the user asks about Carbone tags in Word/LibreOffice headers and footers, why a header tag throws "missing i+1", how to display a loop value in a header or footer, or about floating text boxes used to bridge sections.

---

## Headers, body, and footers are independent sections

In DOCX and ODT templates, the **header**, **body**, and **footer** are three independent sections. Carbone processes tags inside each, but a single Carbone loop must be entirely contained within one section. Placing `[i]` in one section and `[i+1]` in another triggers a `missing i+1` error because Carbone cannot find the pattern to repeat across the section boundary.

What you can put inside a single section (header, body, or footer), without crossing the boundary:

- Root data — `{d.title}`
- Fixed-index access at any depth — `{d.list[0].name}`, `{d.list[0].sub[0].field}`
- Aggregators (print one value, no loop needed) — `{d.list[].field:aggStrD}`, `{d.list[].qty:aggSum}`
- A self-contained loop — both `[i]` and `[i+1]` in the same section

For dynamic per-page headers/footers in **PDF output**, use HTML templates with `<carbone-pdf-header>` / `<carbone-pdf-footer>` instead — see `references/html-templates.md`.

---

## Cross-section loop values — display a body-loop value in the header or footer

When a loop is defined in one section but a tag from that loop needs to appear in another, you have three options. Pick the simplest one that fits.

### Option 1 — aggregator directly in the target section

Works when one combined value (concatenated list, sum, average, count, …) is enough:

```
(header) {d.orders[].lines[].product.name:aggStrD}
```

No loop context is needed because aggregators print exactly one value. The same expression works in the body or footer.

### Option 2 — compute in the body, read anywhere via `c.`

When the same value needs to appear in multiple sections, or when the target section can't host the aggregator chain:

```
(body, anywhere)   {d.orders[].lines[].product.name:aggStrD:set(c.productList)}
(header / footer)  {c.productList}
```

`c.` (complement data) is global and has no loop context, so it works directly in any section. Prefer this over relative `:set` (`...productList`) when the consumer is in a different section — see `references/set-patterns.md` "Relative `:set` dot rule" for the trade-offs.

### Option 3 — floating text box (officially recommended for per-iteration loop tags)

When you need a tag that **keeps its loop context** (so `[i]` resolves correctly) but must visually appear in a section that does not contain the loop, use a floating text box. This is the recommended Carbone pattern for cross-section loop tags.

The core mechanic: the text box **stays anchored in the section where the loop is defined**, but its visual position is moved over the target section. The anchor never crosses the section boundary — only the rendered position does. That way the inner tag inherits the loop's context, while the user sees the value in the header or footer.

Steps (Word):

1. Insert a text box **inside the same section as the loop** (typically the body). Leave the anchor where Word places it — anchored to a paragraph in that section.
2. Right-click the text box → **More Layout Options**:
   - **Horizontal**: Absolute position, to the right of **Page** (set value to slide the box horizontally).
   - **Vertical**: Absolute position, below **Page** (set value to slide the box up into the header band, or down into the footer band).
3. Inside the text box, write whatever Carbone tag needs the loop context — e.g. `{d.orders[i].customer}`, or an aliased / aggregated / `:set`-stored variant depending on what your data already provides. The recipe doesn't require any specific aggregation or alias declaration; it only requires that the tag inside the box references the loop that the anchor belongs to.

**Critical**: do not drag the anchor itself into the header or footer band. Only the visual position is moved. If the anchor ends up in a different section, the loop context is lost and the tag will not resolve.

Caveats:

- The displayed value is whatever the iteration sees on the page where the anchor falls. For a value that should be identical on every page, use a non-partitioned aggregator (Option 1 or 2) instead.
- `:set` → read order is guaranteed only within the same section. If the box reads a `:set` value, keep the producer in the same section.
- Avoid using floating text boxes for decorative content inside loops (set such elements to "In line with text" — see SKILL.md §9). Floating-over-another-section is the supported exception, not the default.
