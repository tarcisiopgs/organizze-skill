# organizze-skill

An [OpenClaw](https://openclaw.ai) / Claude Code skill for the [Organizze](https://www.organizze.com.br) personal finance API.

## Install

```bash
npx skills add tarcisiopgs/organizze-skill@organizze
```

## What it does

Provides authentication patterns, endpoint reference, and code snippets for working with the Organizze REST API v2. Covers:

- Transactions (create, list, filter, update, delete)
- Accounts and credit cards
- Invoices and budget goals
- Categories
- Known API quirks (e.g. installments must be created manually)

## Requirements

Set the `ORGANIZZE_API_KEY` environment variable with your Organizze API token.  
Get yours at: https://app.organizze.com.br/configuracoes/api-keys

## License

MIT
