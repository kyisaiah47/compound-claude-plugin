---
name: redact-pii
description: Strip PII and PHI (names, emails, phones, SSNs, card numbers) from text using the ParseRail kynth_redact tool. Use before storing, logging, sharing, or pasting sensitive text anywhere it should not carry personal data.
---

# Redact PII from text

Use the `kynth_redact` MCP tool. It detects and strips names, emails, phone numbers, SSNs, card numbers, and PHI from text.

## Parameters

- `text` (required), the text to redact
- `types` (optional), an array of entity type names to redact. **Omit it to redact everything detected.** Pass it only when the user wants some categories kept, for example redact SSNs and cards but keep names.
- `placeholder` (optional), the replacement token. Omit for the default.

## Workflow

1. If the sensitive text is in a file, read it and pass the contents as `text`. For a PDF or an image, extract the text first with `kynth_parse`, then redact the result.
2. Call `kynth_redact` with the text. Default to full redaction, with no `types`, unless the user scoped it.
3. Return the redacted text. Check it against anything the user specifically flagged; if a flagged item survived, run a second pass with an explicit `types` entry for it.
4. Never echo the original unredacted values back in your summary. The point is that they stop propagating.

## Billing

3 credits per successful call, 1 credit = $0.01, debited from the account wallet only on success. Failed calls cost nothing.

There is no free tier. A wallet starts at zero and is topped up at https://api.thecompound.tech. An uncovered call returns `402 insufficient_credits` before any model runs.
