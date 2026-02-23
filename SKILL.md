---
name: organizze
description: Access and manage the Organizze personal finance API. Use when working with Organizze to manage transactions, accounts, credit cards, invoices, transfers, categories, or budgets. Triggers on any request involving financial records, expenses, income, installments, recurring transactions, credit card invoices, transfers, or balance queries in Organizze.
---

# Organizze API

## Authentication

Basic auth with `User-Agent` header (**required** — returns 400 without it):

```python
import os, base64, urllib.request, json

api_key = os.environ.get('ORGANIZZE_API_KEY', '')
email = 'user@example.com'  # Organizze account email
credentials = base64.b64encode(f'{email}:{api_key}'.encode()).decode()
headers = {
    'Authorization': f'Basic {credentials}',
    'Content-Type': 'application/json',
    'User-Agent': 'myapp/1.0'
}
BASE = 'https://api.organizze.com.br/rest/v2'
```

## Key Conventions

- **Amounts**: always in cents (`amount_cents`). Negative = expense/debit, positive = income/credit.
- **Pagination**: `/transactions` and `/invoices` default to current month/year. Use `?start_date=YYYY-MM-DD&end_date=YYYY-MM-DD`. Dates are rounded to full months.
- **Credit card transactions**: have `credit_card_id` set; bank account transactions have it as `null`.
- **Transfers**: use `/transfers` endpoint, not `/transactions`. Creates two linked records (`oposite_transaction_id`).

## Endpoint Map

| Resource | Endpoints |
|---|---|
| Accounts | `GET/POST /accounts`, `GET/PUT/DELETE /accounts/{id}` |
| Budgets | `GET /budgets`, `/budgets/{year}`, `/budgets/{year}/{month}` |
| Categories | `GET/POST /categories`, `GET/PUT/DELETE /categories/{id}` |
| Credit Cards | `GET/POST /credit_cards`, `GET/PUT/DELETE /credit_cards/{id}` |
| Invoices | `GET /credit_cards/{id}/invoices`, `GET /credit_cards/{id}/invoices/{id}` |
| Invoice Payments | `GET /credit_cards/{id}/invoices/{id}/payments` |
| Transactions | `GET/POST /transactions`, `GET/PUT/DELETE /transactions/{id}` |
| Transfers | `GET/POST /transfers`, `GET/PUT/DELETE /transfers/{id}` |
| Users | `GET /users/{id}` |

Full schemas and response examples: see `references/api_reference.md`.

## Common Patterns

### List transactions for a period
```python
req = urllib.request.Request(
    f'{BASE}/transactions?start_date=2026-02-01&end_date=2026-02-28',
    headers=headers
)
with urllib.request.urlopen(req) as r:
    txs = json.loads(r.read())
```

### Create a simple transaction
```python
body = json.dumps({
    'description': 'Mercado',
    'date': '2026-03-01',
    'amount_cents': -5000,
    'paid': False,
    'account_id': 123,
    'category_id': 456,
    'notes': 'Compras da semana',
    'tags': [{'name': 'alimentacao'}]
}).encode()
req = urllib.request.Request(f'{BASE}/transactions', data=body, headers=headers, method='POST')
```

### Create installment transaction (correct way)
```python
# Use installments_attributes — NOT total_installments (ignored by API)
body = json.dumps({
    'description': 'Notebook',
    'date': '2026-02-01',
    'amount_cents': -150000,
    'paid': False,
    'credit_card_id': 789,
    'category_id': 456,
    'installments_attributes': {'periodicity': 'monthly', 'total': 12}
}).encode()
req = urllib.request.Request(f'{BASE}/transactions', data=body, headers=headers, method='POST')
# Creates 12 monthly installment records automatically
```

### Create recurring (fixed) transaction
```python
body = json.dumps({
    'description': 'Aluguel',
    'date': '2026-03-05',
    'amount_cents': -150000,
    'paid': False,
    'account_id': 123,
    'category_id': 456,
    'recurrence_attributes': {'periodicity': 'monthly'}
}).encode()
# periodicity options: monthly, yearly, weekly, biweekly, bimonthly, trimonthly
```

### Create a transfer between accounts
```python
body = json.dumps({
    'debit_account_id': 111,   # source account (money leaves)
    'credit_account_id': 222,  # destination account (money arrives)
    'amount_cents': 50000,
    'date': '2026-03-01',
    'paid': True
}).encode()
req = urllib.request.Request(f'{BASE}/transfers', data=body, headers=headers, method='POST')
# Credit cards are NOT accepted as transfer source or destination
```

### Update/delete recurring or installment transactions
```python
# Update this and future occurrences
body = json.dumps({'amount_cents': -160000, 'update_future': True}).encode()
# Update ALL occurrences (including past — may affect balance)
body = json.dumps({'amount_cents': -160000, 'update_all': True}).encode()

req = urllib.request.Request(f'{BASE}/transactions/97', data=body, headers=headers, method='PUT')
```

### Check invoices due on a specific date
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
            pending = abs(inv['balance_cents']) / 100
            print(f'{card["name"]}: R$ {pending:.2f} pendente')
```
