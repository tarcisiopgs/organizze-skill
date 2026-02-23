# organizze-skill

An agent skill for the [Organizze](https://www.organizze.com.br) personal finance API. Compatible with any agentic tool that supports the [skills.sh](https://skills.sh) ecosystem.

## Install

```bash
npx skills add tarcisiopgs/organizze-skill
```

## What it does

Provides authentication patterns, endpoint reference, and ready-to-use code snippets for the Organizze REST API v2. Covers:

- Transactions (create, list, filter, update, delete)
- Accounts and credit cards
- Invoices and budget goals
- Categories
- Known API quirks (e.g. installments must be created manually per transaction)

## Requirements

Set the `ORGANIZZE_API_KEY` environment variable with your Organizze API token.  
Get yours at: https://app.organizze.com.br/configuracoes/api-keys

## License

MIT
