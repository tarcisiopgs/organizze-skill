# organizze-skill

An agent skill for the [Organizze](https://organizze.com.br) personal finance API — compatible with any skills.sh-enabled assistant.

## What it does

Once installed, this skill lets you manage your Organizze account via natural language:

- Transactions: list, create, update, delete (including installments and recurring)
- Accounts and credit cards management
- Invoices and invoice payments
- Transfers between accounts
- Categories and budgets
- Users

## Setup

Set your Organizze credentials as environment variables:

```bash
export ORGANIZZE_API_KEY=your-api-key-here
export ORGANIZZE_EMAIL=your-email@example.com
```

## Usage examples

```
list my transactions for March 2026
create a monthly recurring expense of R$1500 for rent
show my credit card invoices
how much do I owe on my credit cards this month?
transfer R$500 from checking to savings
```

## Installation

```bash
claude install tarcisiopgs/organizze-skill
```

## API Coverage

Covers the full Organizze REST API v2:

- Accounts, Credit Cards, Invoices, Invoice Payments
- Transactions (one-time, installment, recurring)
- Transfers, Categories, Budgets, Users

## License

MIT
