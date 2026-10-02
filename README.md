# ParseRail for Claude Code

This [Claude Code](https://code.claude.com) plugin puts document work in the session through [ParseRail](https://parserail.thecompound.tech). Installing the plugin configures the [`parserail-mcp`](https://www.npmjs.com/package/parserail-mcp) MCP server with 41 tools and adds three skills that teach Claude how to use it:

| Skill | What it does |
| --- | --- |
| `extract-financial-doc` | Routes invoices, receipts, bank and card statements, resumes, contracts and tables through the right ParseRail endpoint and returns schema-validated JSON: vendor, line items, totals, normalized transactions |
| `redact-pii` | Strips names, emails, phones, SSNs and card numbers from text before it gets stored, logged, or shared |
| `parserail-api` | The reference: auth, the credit wallet, error shapes, and which of the 41 tools fits which job, with per-call costs |

ParseRail is the developer platform from [Compound Labs](https://thecompound.tech). It parses documents, extracts fields, redacts PII, analyzes contracts, fights chargebacks and enriches companies, each capability solved once and running in production behind one API key and one pay-per-call credit wallet.

## Install

Run these commands inside a Claude Code session. This repository also serves as its own marketplace:

```
/plugin marketplace add kyisaiah47/parserail-claude-plugin
/plugin install parserail@compound
```

Or from your shell:

```bash
claude plugin marketplace add kyisaiah47/parserail-claude-plugin
claude plugin install parserail@compound
```

Installed before the 0.2.0 rename? The marketplace carries a `renames` entry, so `compound-core` migrates to `parserail` on its own.

## Set your API key

Create a key at **[api.thecompound.tech](https://api.thecompound.tech)**. Keys start `ksk_live_`. Make the key available to the MCP server:

```bash
export PARSERAIL_API_KEY=ksk_live_your_key_here
```

Keep the key in your shell profile so it survives restarts. The server reads the key once at startup and exits with a message if the key is missing.

You pay per call from a credit wallet at 1 credit = $0.01, and **only successful calls are charged**. Errors and retries cost nothing. There is no free tier: a wallet starts at zero and you top it up with a credit pack or a monthly plan in the dashboard.

## Use it

```
> pull the line items out of ./invoices/acme-march.pdf
> redact the PII from this support transcript
> what's my parserail credit balance?
```

The plugin namespaces these skills: `/parserail:extract-financial-doc`, `/parserail:redact-pii`, `/parserail:parserail-api`. Claude also invokes a skill on its own when a task matches.

## Requirements

- You need Node.js 18 or newer. The MCP server runs via `npx`.
- You need a ParseRail API key in `PARSERAIL_API_KEY`.

## Also available

- **Gemini CLI**: [parserail-gemini-extension](https://github.com/kyisaiah47/parserail-gemini-extension)
- Any MCP client can run `npx -y parserail-mcp`. The official MCP registry lists the package as `tech.thecompound/parserail-mcp`.
- **Keyless lookups from Compound Labs**: [OpenLookup](https://github.com/kyisaiah47/openlookup) (`npx -y openlookup`) provides eleven read-only tools over live public data. You need no API key or signup.

## License

MIT
