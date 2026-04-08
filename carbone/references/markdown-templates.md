# Carbone Markdown Templates Reference

Version: 5.0+ | Free feature
Output formats: PDF, DOCX, ODT, PNG, JPG, MD

Markdown templates support substitutions, repetitions, formatters, translations, conditions (inline + blocks), and all enterprise features. Ideal for AI-generated content workflows (LLMs natively output Markdown).

## Contents

- [Basics](#basics)
- [Loops](#loops)
- [Conditions](#conditions)
  - [Inline conditions](#inline-conditions)
  - [Conditional sections — `:showBegin` / `:showEnd`](#conditional-sections--showbegin--showend)
  - [Conditional sections — `:hideBegin` / `:hideEnd`](#conditional-sections--hidebegin--hideend)
  - [`:drop` / `:keep` via HTML blocks](#drop--keep-via-html-blocks)
- [Pictures](#pictures)
- [Hyperlinks](#hyperlinks)
- [Formatters](#formatters-same-as-all-other-formats)
- [i18n / Translations](#i18n--translations)
- [Comments](#comments)
- [Limitations](#limitations-features-not-yet-supported-in-markdown)
- [Convert to PDF — API call](#convert-to-pdf--api-call)

---

## Basics

Write any valid Markdown. Carbone tags work inline anywhere in the text:

```markdown
# Welcome, {d.user.firstName} {d.user.lastName}!

Your email: {d.user.email}

Membership: {d.membershipLevel}
```

---

## Loops

**Loops in Markdown are only supported inside tables.** Place the loop tags inside table cells, with the `[i+1]` end marker in the next row (which is removed from output).

```markdown
| ID | Name | Role |
| -- | ---- | ---- |
| {d.users[i].id} | {d.users[i].name} | {d.users[i].role} |
| {d.users[i+1]} | | |
```

---

## Conditions

### Inline conditions

Use `:ifEQ`, `:show`, `:elseShow` directly inside Markdown text:

```markdown
Status: {d.status:ifEQ('active'):show('✅ Active'):elseShow('❌ Inactive')}

Score: {d.score:ifGT(90):show('Excellent'):elseShow('Good')}

Priority: {d.isUrgent:ifEQ(true):show('**HIGH**'):elseShow('Normal')}
```

### Conditional sections — `:showBegin` / `:showEnd`

```markdown
{d.showDetails:ifEQ(true):showBegin}
## Details

- Feature 1
- Feature 2

{d.showDetails:ifEQ(true):showEnd}
```

### Conditional sections — `:hideBegin` / `:hideEnd`

```markdown
{d.name:ifEM:hideBegin}
Welcome, **{d.name}**!
{d.name:ifEM:hideEnd}
```

### `:drop` / `:keep` via HTML blocks

⚠️ **Coming soon** — Not yet supported in Markdown templates.

---

## Pictures

Use standard Markdown image syntax with a Carbone tag as the `src`. Supports absolute URLs and Base64 Data-URIs. **Relative paths are not supported.**

```markdown
![Alt text]({d.imageUrl})
![Profile]({d.user.profilePictureBase64})
```

---

## Hyperlinks

Use standard Markdown link syntax with Carbone tags in the URL and/or label:

```markdown
[Read the Docs]({d.documentationUrl})

[{d.isLoggedIn:ifEQ(true):show('Your Profile'):elseShow('Login')}]({d.isLoggedIn:ifEQ(true):show(d.profileUrl):elseShow(d.defaultUrl)})

[Learn about {d.product.name}]({d.product.url})

[Documentation](/docs/{d.product.name:lowerCase:replace(' ','-')})
```

---

## Formatters (same as all other formats)

All standard Carbone formatters work in Markdown templates:

```markdown
Order date: {d.orderDate:formatD('DD/MM/YYYY')}
Delivery: {d.deliveryDate:formatD('dddd DD MMMM YYYY')}
Quantity: {d.quantity:formatN(2)}
Price: {d.price:formatC(2,'EUR')}
```

---

## i18n / Translations

Same mechanism as HTML templates — pass `translations` + `lang` in the API request:

```markdown
# {t(Invoice)}

{t(Date)}: {d.date:formatD('DD/MM/YYYY')}

| Item | Quantity |
| ---- | -------- |
| {d.tool:t} | 1 |
| {d.protection:t} | 2 |
```

---

## Comments

Carbone processes tags inside HTML comments in Markdown:
```markdown
<!-- {d.user.name} -->   ← replaced with data value
```
To escape: break the tag syntax — `<!-- d.user.name -->`.

---

## Limitations (features not yet supported in Markdown)

| Feature | Status |
|---|---|
| `:drop` / `:keep` via HTML blocks | ⚠️ Coming soon |
| Custom CSS styling | ⚠️ Coming soon |
| Dynamic barcodes | ⚠️ Coming soon |
| Inject HTML in Markdown (`:html`) | ⚠️ Coming soon |
| Page breaks | ⚠️ Coming soon |
| Table of Contents | ⚠️ Coming soon |
| PDF conversion options (paper size, margins, headers, footers) | ⚠️ Not supported — use HTML templates instead |

---

## Convert to PDF — API call

```bash
curl --request POST 'https://api.carbone.io/render/template?download=true' \
  --header 'carbone-version: 5' \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer API_TOKEN' \
  --data-raw '{
    "data": {},
    "template": "<BASE64_ENCODED_MARKDOWN>",
    "convertTo": "pdf",
    "converter": "L"
  }' \
  --output result.pdf
```

`converter: "L"` = LibreOffice engine (use for Markdown → PDF/DOCX/ODT).
Supported `convertTo` values: `pdf`, `docx`, `odt`, `jpg`, `png`, `md`.

> For advanced PDF layout control (paper size, margins, headers, footers), use an HTML template instead.
