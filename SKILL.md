---
name: organizze
description: Access and manage the Organizze personal finance API. Use when working with the Organizze app to manage transactions, accounts, credit cards, invoices, categories, or budgets. Triggers on any request involving financial records, expenses, income, credit card invoices, or balance queries in Organizze.
---

# Organizze API

## Authentication

Basic auth with `User-Agent` header (required — returns 400 without it):

```python
import os, base64, urllib.request, json

api_key = os.environ.get('ORGANIZZE_API_KEY', '')
email = 'tarcisiopgs@gmail.com'
credentials = base64.b64encode(f'{email}:{api_key}'.encode()).decode()
headers = {
    'Authorization': f'Basic {credentials}',
    'Content-Type': 'application/json',
    'User-Agent': 'estagisaura/1.0'
}
BASE = 'https://api.organizze.com.br/rest/v2'
```

## Key Conventions

- **Amounts**: always in cents (`amount_cents`). Negative = expense, positive = income.
- **Pagination**: transactions and invoices require `?start_date=YYYY-MM-DD&end_date=YYYY-MM-DD`. Default = current month/year.
- **Credit card transactions**: have `credit_card_id` set; regular account transactions have it as `null`.
- **Installments**: the API **ignores** `total_installments` in POST — create each installment as a separate transaction with monthly dates and descriptions like "Descrição 1/N", "Descrição 2/N".

## Quick Reference

| Resource | Endpoint |
|---|---|
| Accounts | `GET/POST /accounts`, `GET/PUT/DELETE /accounts/{id}` |
| Categories | `GET/POST /categories`, `GET/PUT/DELETE /categories/{id}` |
| Credit Cards | `GET/POST /credit_cards`, `GET/PUT/DELETE /credit_cards/{id}` |
| Invoices | `GET /credit_cards/{id}/invoices`, `GET /credit_cards/{id}/invoices/{id}` |
| Transactions | `GET/POST /transactions`, `GET/PUT/DELETE /transactions/{id}` |
| Budgets | `GET /budgets`, `GET /budgets/{year}`, `GET /budgets/{year}/{month}` |
| Users | `GET /users/{id}` |

Full endpoint details and response schemas: see `references/api_reference.md`.

## Common Patterns

### List pending non-credit-card transactions for a date
```python
req = urllib.request.Request(f'{BASE}/transactions?start_date={date}&end_date={date}', headers=headers)
with urllib.request.urlopen(req) as r:
    txs = json.loads(r.read())
pending = [t for t in txs if not t['paid'] and t['credit_card_id'] is None]
```

### Create a transaction
```python
body = json.dumps({
    'description': 'Mercado',
    'date': '2026-03-01',
    'amount_cents': -5000,       # negative = expense
    'paid': False,
    'account_id': 6964391,       # account or credit_card_id
    'category_id': 149849081
}).encode()
req = urllib.request.Request(f'{BASE}/transactions', data=body, headers=headers, method='POST')
```

### Create installments manually (API limitation)
```python
from datetime import date
from dateutil.relativedelta import relativedelta

start = date(2026, 2, 23)
total = 2
amount_per = -9037  # R$90,37
for i in range(total):
    d = start + relativedelta(months=i)
    body = json.dumps({
        'description': f'Shein {i+1}/{total}',
        'date': d.isoformat(),
        'amount_cents': amount_per,
        'paid': False,
        'credit_card_id': 2221313,
        'category_id': 153598684
    }).encode()
    req = urllib.request.Request(f'{BASE}/transactions', data=body, headers=headers, method='POST')
    with urllib.request.urlopen(req) as r:
        print(json.loads(r.read())['id'])
```

### Check credit card invoices due on a date
```python
req = urllib.request.Request(f'{BASE}/credit_cards', headers=headers)
with urllib.request.urlopen(req) as r:
    cards = json.loads(r.read())

for card in cards:
    req = urllib.request.Request(f'{BASE}/credit_cards/{card["id"]}/invoices', headers=headers)
    with urllib.request.urlopen(req) as r:
        invoices = json.loads(r.read())
    for inv in invoices:
        if inv['date'] == target_date and inv.get('balance_cents', 0) != 0:
            print(f'{card["name"]}: R$ {abs(inv["balance_cents"])/100:.2f} pendente')
```
