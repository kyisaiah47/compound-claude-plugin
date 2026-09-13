---
name: extract-financial-doc
description: Extract structured data from a financial document (invoice, receipt, bank or card statement, resume, contract, or embedded tables) using the ParseRail MCP tools. Use when the user wants line items, totals, transactions, or tables pulled out of a PDF, an image, or pasted text as clean JSON.
---

# Extract a financial document

Route the document through the ParseRail tool that matches its type. The specialized tools return richer, schema-validated JSON and most of them cost fewer credits than generic parsing.

## 1. Pick the tool

| Document | Tool | Output | Credits |
| --- | --- | --- | --- |
| Invoice | `compound_invoice` | vendor, dates, PO refs, tax, totals, line items | 8 |
| Receipt | `compound_receipt` | merchant, items, totals, payment method, expense category | 6 |
| Bank or card statement | `compound_statement` | account, period, balances, every transaction as a normalized row | 12 |
| Tables in any document, scanned included | `compound_tables` | every table as clean headers and rows | 8 |
| Resume or CV | `compound_resume` | contact, skills, experience, education, links | 8 |
| Contract | `compound_contract` | parties, effective date, term, renewal, governing law, obligations, risk clauses | 12 |
| Unknown or other (EOB, ERA, COI) | `compound_parse` | structured fields, with an optional `docType` hint such as `"invoice"` | 10 |
| Multi-document scan bundle | `compound_split` | what each document is and where it starts and ends, then extract each piece | 10 |
| Two versions of one document | `compound_compare` | every material change, what it means, the risk it carries | 15 |

If the user needs the output in their own shape, run the extraction tool first, then pass the result through `compound_structure` with their JSON Schema. It validates against required fields and property types, retries once correctively, and returns a `valid` flag. That is 6 more credits.

## 2. Pass the document

Every tool above takes exactly ONE of these input forms:

- `fileUrl`, a public URL to the PDF or image
- `text`, raw text if the content is already in the conversation
- `fileBase64` plus `fileMimeType`, for local files: read the file, base64-encode it, and pass both fields. `fileMimeType` has to be right, for example `application/pdf`, `image/png`, `image/jpeg`.

For a local file, encode it in one step:

```bash
base64 -i path/to/invoice.pdf
```

Do not pass a local filesystem path as `fileUrl`. That field has to be publicly reachable. Passing none of the three forms returns `Provide one of: fileUrl, text, or fileBase64 + fileMimeType.`

## 3. What comes back

Each tool returns schema-validated JSON for its document type. The API validates the extraction against the endpoint's schema before returning it, so fields are typed and consistent across calls. Alongside the result comes `usage: { credits, balanceRemaining }`.

Present the parts the user asked for. Offer the full JSON if they want to pipe it somewhere.

## Billing

Calls debit the account credit wallet only on success. A failed call (bad input, timeout, server error) costs nothing, so a retry is safe. 1 credit = $0.01; the per-tool rates are in the table above.

There is no free tier. A wallet starts at zero and is topped up by a credit pack or a monthly plan bought at https://api.thecompound.tech. A call that the wallet cannot cover returns `402 insufficient_credits` before any model runs, so it costs nothing and answers immediately.
