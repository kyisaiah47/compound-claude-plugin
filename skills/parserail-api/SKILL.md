---
name: parserail-api
description: Reference for the ParseRail API and its MCP tools. Covers authentication, the credit wallet, error shapes, and which of the 41 tools fits which job. Use when picking a ParseRail tool, debugging a 401/402/503, or answering a question about ParseRail pricing or capabilities.
---

# ParseRail API reference

ParseRail (https://parserail.thecompound.tech, API host https://api.thecompound.tech) turns documents and messy text into schema-validated JSON. This plugin exposes it as MCP tools under the server `parserail`, via the `@compound/api-mcp` package.

## Authentication

- The MCP server reads the key from the `COMPOUND_API_KEY` environment variable. Keys start `ksk_live_`.
- Mint a key in the dashboard at https://api.thecompound.tech.
- On the wire it is `Authorization: Bearer ksk_live_…`, a header the MCP server sets itself.
- Tools failing with `401 unauthorized` mean the key is missing or mistyped. Export `COMPOUND_API_KEY` and restart Claude Code. The server reads the variable once at startup and exits immediately if it is absent.

## Credits

- 1 credit = $0.01. Each capability call debits the account wallet at that endpoint's rate, and only on success. A failed call is not charged, so retrying is safe.
- **There is no free tier.** A new wallet starts at zero and nothing tops it up. Credits come from a one-time pack or a monthly plan bought in the dashboard, and they do not expire.
- A successful call returns its result plus `usage: { credits, balanceRemaining }`.
- `compound_account` returns the current balance. It costs nothing.
- `402 insufficient_credits` means the wallet cannot cover the call. The balance check runs before any model is touched, so an empty wallet costs nothing and answers immediately. Buy credits at https://api.thecompound.tech.

## Errors

The API returns `{ "error": { "code", "message" }, "requestId" }`. The MCP tools surface it as `ParseRail error [code] (HTTP status): message`. The codes seen most: `unauthorized` (401), `insufficient_credits` (402), `rate_limited` (429), `server_error` (500). Every error carries a request id.

Each endpoint has its own per-key rate window. Most sit between 60 and 300 calls a minute; `research` is 10. A `429` clears on its own inside the window.

## Document input

The document tools take exactly one of:

- `fileUrl`, a public URL to a PDF or image
- `text`, raw text when the content is already in the conversation
- `fileBase64` plus `fileMimeType`, local file bytes base64-encoded with the correct MIME type (`application/pdf`, `image/png`, `image/jpeg`)

Passing none of the three returns `Provide one of: fileUrl, text, or fileBase64 + fileMimeType.` A local filesystem path is not a `fileUrl`.

## Tools and costs (credits per successful call)

### Documents
| Tool | Does | Cost |
| --- | --- | --- |
| `compound_parse` | any document into structured fields | 10 |
| `compound_invoice` | invoice into vendor, dates, PO refs, tax, totals, line items | 8 |
| `compound_receipt` | receipt into merchant, items, totals, payment method, expense category | 6 |
| `compound_statement` | bank or card statement into normalized transactions | 12 |
| `compound_tables` | every table in a document into headers and rows | 8 |
| `compound_resume` | resume or CV into a structured candidate profile | 8 |
| `compound_contract` | contract into parties, term, renewal, obligations, risk clauses | 12 |
| `compound_split` | multi-document scan bundle into classified segments | 10 |
| `compound_compare` | two document versions into material changes and risk | 15 |

### Text and data
| Tool | Does | Cost |
| --- | --- | --- |
| `compound_extract` | pull a field set you name out of text | 4 |
| `compound_structure` | messy input plus YOUR JSON Schema into validated output | 6 |
| `compound_redact` | strip PII and PHI from text | 3 |
| `compound_classify` | label one text against your taxonomy | 2 |
| `compound_categorize` | label up to a hundred items in one call | 3 |
| `compound_summarize` | summary, key points, action items | 3 |
| `compound_normalize` | messy records into canonical rows plus a change log | 5 |
| `compound_match` | entity resolution across two record sets | 8 |
| `compound_sentiment` | sentiment, aspects, themes | 2 |
| `compound_moderate` | text or image against your policy: allow, review, block | 2 |

### Commerce and ops
| Tool | Does | Cost |
| --- | --- | --- |
| `compound_chargeback` | dispute into a representment packet | 15 |
| `compound_po_match` | invoice vs PO vs receipt, three-way match | 10 |
| `compound_fraud_flag` | order or transaction into a risk score and checks | 6 |
| `compound_dunning` | overdue invoice into a collection sequence | 8 |
| `compound_quote` | job plus your rates into an itemized quote | 8 |
| `compound_late_fee_rules` | the maximum late-fee rate a US state allows on a B2B invoice, with the statute cite | free |

### Communication and growth
| Tool | Does | Cost |
| --- | --- | --- |
| `compound_reply` | thread into a ready-to-send reply | 5 |
| `compound_triage` | ticket into priority, category, team, SLA risk | 3 |
| `compound_minutes` | transcript into minutes, decisions, action items | 6 |
| `compound_review_reply` | review into a public response | 3 |
| `compound_outreach` | enriched lead into an outreach sequence | 6 |
| `compound_enrich` | domain or work email into a company profile | 5 |
| `compound_research` | company or topic into a cited multi-source brief | 50 |
| `compound_screen` | lead vs your ICP into a qualification verdict | 10 |
| `compound_rewrite` | copy into your brand voice | 3 |
| `compound_product_copy` | specs into listing-ready copy | 5 |

### Media and agents
| Tool | Does | Cost |
| --- | --- | --- |
| `compound_describe` | image into alt text, caption, tags, embedded text | 3 |
| `compound_image` | prompt into a generated image | 10 |
| `compound_speak` | text into speech audio | 5 |
| `compound_memory` | agent memory: store, search, forget | 2 |
| `compound_transcribe` | audio into text with speakers and timestamps | paused |
| `compound_account` | credit balance | free |

`compound_transcribe` has returned 503 since 2026-08-19 and is never charged. It is paused until a multimodal provider is available under a no-training agreement, because a customer recording is customer content in the same way an invoice is. Do not route audio through it. Say it is unavailable.

## Choosing between overlapping tools

- A known document type takes the specialized tool (`compound_invoice`, `compound_receipt`, `compound_statement`, `compound_resume`, `compound_contract`). An unknown one takes `compound_parse` with an optional `docType` hint.
- Output in the user's own shape: run the extraction tool first, then pass its result through `compound_structure` with their JSON Schema. That tool validates against required fields and property types, retries once correctively, and returns a `valid` flag.
- One text and a few labels: `compound_classify`. Many items at once: `compound_categorize`, up to a hundred.
- Fields out of plain text: `compound_extract`. Fields out of a PDF or image: a document tool.
