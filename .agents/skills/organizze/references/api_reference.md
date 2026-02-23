# Organizze API Reference

Base URL: `https://api.organizze.com.br/rest/v2`
Auth: HTTP Basic (`email:api_token`). Header `User-Agent` required.
API key URL: https://app.organizze.com.br/configuracoes/api-keys

---

## Users

```
GET /users/{id}
```
Response: `{ id, name, email, role }`

---

## Accounts

```
GET    /accounts
GET    /accounts/{id}
POST   /accounts        body: { name, type, description, default }
PUT    /accounts/{id}   body: { name, ... }
DELETE /accounts/{id}
```

Account types: `checking`, `savings`, `other`

Response fields: `id, name, description, archived, created_at, updated_at, default, type`

---

## Categories

```
GET    /categories
GET    /categories/{id}
POST   /categories      body: { name }
PUT    /categories/{id} body: { name }
DELETE /categories/{id} body: { replacement_id }  ← optional, reassigns transactions
```

Response fields: `id, name, color, parent_id`

`parent_id: null` = top-level category. Sub-categories have a `parent_id`.

---

## Credit Cards

```
GET    /credit_cards
GET    /credit_cards/{id}
POST   /credit_cards      body: { name, card_network, due_day, closing_day, limit_cents }
PUT    /credit_cards/{id} body: { name, due_day, closing_day, update_invoices_since }
DELETE /credit_cards/{id}
```

Response fields: `id, name, description, card_network, closing_day, due_day, limit_cents, kind, archived, default, created_at, updated_at`

---

## Invoices

```
GET /credit_cards/{id}/invoices
GET /credit_cards/{id}/invoices/{invoice_id}
```

Response fields: `id, date, starting_date, closing_date, amount_cents, payment_amount_cents, balance_cents, previous_balance_cents, credit_card_id`

Detailed invoice also includes `transactions[]`.

**Pending invoice check**: `date == target_date AND balance_cents != 0`

**Pagination**: defaults to current year. Add `?start_date=YYYY-MM-DD&end_date=YYYY-MM-DD` for range.

---

## Transactions

```
GET    /transactions?start_date=YYYY-MM-DD&end_date=YYYY-MM-DD
GET    /transactions/{id}
POST   /transactions      body: (see below)
PUT    /transactions/{id} body: (partial update supported)
DELETE /transactions/{id}
```

### POST body fields

| Field | Type | Description |
|---|---|---|
| `description` | string | Required |
| `date` | string | `YYYY-MM-DD` |
| `amount_cents` | int | Negative = expense, positive = income |
| `paid` | bool | Whether already paid/received |
| `account_id` | int | Bank account ID (for regular transactions) |
| `credit_card_id` | int | Credit card ID (for card transactions) |
| `category_id` | int | Category ID |
| `notes` | string | Optional notes |
| `tags` | array | Optional tags |

**Note:** `total_installments` is documented but **ignored** by the API. Create installments manually as separate transactions.

### Response fields

`id, description, date, paid, amount_cents, total_installments, installment, recurring, account_id, category_id, notes, attachments_count, credit_card_id, credit_card_invoice_id, paid_credit_card_id, paid_credit_card_invoice_id, oposite_transaction_id, oposite_account_id, created_at, updated_at, tags, attachments, recurrence_id`

### Useful filters (client-side)

```python
# Pending non-card expenses
[t for t in txs if not t['paid'] and t['credit_card_id'] is None and t['amount_cents'] < 0]

# Pending non-card income
[t for t in txs if not t['paid'] and t['credit_card_id'] is None and t['amount_cents'] > 0]

# All pending (no card)
[t for t in txs if not t['paid'] and t['credit_card_id'] is None]

# Card transactions only
[t for t in txs if t['credit_card_id'] is not None]
```

---

## Budgets (Metas)

```
GET /budgets               # current month
GET /budgets/{year}        # full year
GET /budgets/{year}/{month}
```

Response: `[{ amount_in_cents, category_id, date, activity_type, total, predicted_total, percentage }]`

---

## Error Codes

| Status | Meaning |
|---|---|
| 400 | Bad request (usually missing User-Agent header) |
| 401 | Invalid email or API token |
| 422 | Validation error (check `errors` field in response) |
