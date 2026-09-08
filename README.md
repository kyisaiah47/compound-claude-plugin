# ParseRail for Claude Code

A [Claude Code](https://code.claude.com) plugin that puts document work in the session, powered by [ParseRail](https://parserail.kynth.studio). Installing it wires up the [`@kynth/api-mcp`](https://www.npmjs.com/package/@kynth/api-mcp) MCP server (41 tools) and adds three skills that teach Claude how to use it:

| Skill | What it does |
| --- | --- |
| `extract-financial-doc` | Routes invoices, receipts, bank and card statements, resumes, contracts and tables through the right ParseRail endpoint and returns schema-validated JSON: vendor, line items, totals, normalized transactions |
| `redact-pii` | Strips names, emails, phones, SSNs and card numbers from text before it gets stored, logged, or shared |
| `parserail-api` | The reference: auth, the credit wallet, error shapes, and which of the 41 tools fits which job, with per-call costs |

ParseRail is the developer platform from [Compound Labs](https://thecompound.tech). It parses documents, extracts fields, redacts PII, analyzes contracts, fights chargebacks and enriches companies, each capability solved once and running in production behind one API key and one pay-per-call credit wallet.

## Install

Inside a Claude Code session. This repo doubles as its own marketplace:

```
/plugin marketplace add kyisaiah47/kynth-claude-plugin
/plugin install parserail@kynth
```

Or from your shell:

```bash
claude plugin marketplace add kyisaiah47/kynth-claude-plugin
claude plugin install parserail@kynth
```

Installed before the 0.2.0 rename? The marketplace carries a `renames` entry, so `kynth-core` migrates to `parserail` on its own.

## Set your API key

Mint a key at **[api.kynth.studio](https://api.kynth.studio)**. Keys start `ksk_live_`. Make it available to the MCP server:

```bash
export KYNTH_API_KEY=ksk_live_your_key_here
```

Put it in your shell profile so it survives restarts. The server reads it once at startup and exits with a message if it is missing.

Billing is pay-per-call from a credit wallet at 1 credit = $0.01, and **only successful calls are charged**. Errors and retries cost nothing. There is no free tier: a wallet starts at zero and is topped up with a credit pack or a monthly plan in the dashboard.

## Use it

```
> pull the line items out of ./invoices/acme-march.pdf
> redact the PII from this support transcript
> what's my parserail credit balance?
```

Skills are namespaced under the plugin: `/parserail:extract-financial-doc`, `/parserail:redact-pii`, `/parserail:parserail-api`. Claude also invokes them on its own when a task matches.

## Requirements

- Node.js 18 or newer. The MCP server runs via `npx`.
- A ParseRail API key in `KYNTH_API_KEY`.

## Also available

- **Gemini CLI**: [kynth-gemini-extension](https://github.com/kyisaiah47/kynth-gemini-extension)
- **Any MCP client**: `npx -y @kynth/api-mcp`, listed in the official MCP registry as `studio.kynth/core`
- **Keyless lookups from Compound Labs**: [kynth-mcp](https://github.com/kyisaiah47/kynth-mcp), eleven read-only tools over live public data, no key and no signup

## License

MIT
