# Organizze API Reference

Base URL: `https://api.organizze.com.br/rest/v2`  
Auth: HTTP Basic (`email:api_token`). Header `User-Agent` **required** (400 without it).  
API key: https://app.organizze.com.br/configuracoes/api-keys

---

## Pagination

Transactions and invoices default to the **current month/year** if no period is given.  
Use `?start_date=YYYY-MM-DD&end_date=YYYY-MM-DD`. Dates are rounded to full months.  
Filter transactions by account: `?account_id={id}`.

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

Account `type` values: `checking`, `savings`, `other`  
Response fields: `id, name, description, archived, created_at, updated_at, default, type`

---

## Categories

```
GET    /categories
GET    /categories/{id}
POST   /categories      body: { name }
PUT    /categories/{id} body: { name }
DELETE /categories/{id} body: { replacement_id }  ← reassigns transactions before deleting
```

Response fields: `id, name, color, parent_id`  
`parent_id: null` = top-level category.

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
GET /credit_cards/{id}/invoices                        (defaults to current year)
GET /credit_cards/{id}/invoices/{invoice_id}           (includes transactions[] and payments[])
GET /credit_cards/{id}/invoices/{invoice_id}/payments
```

Response fields: `id, date, starting_date, closing_date, amount_cents, payment_amount_cents, balance_cents, previous_balance_cents, credit_card_id`

Detailed invoice also includes:
- `transactions[]` — purchases in that invoice period
- `payments[]` — payment records

**Pending invoice**: `date == target_date AND balance_cents != 0`

---

## Transactions

```
GET    /transactions?start_date=YYYY-MM-DD&end_date=YYYY-MM-DD[&account_id={id}]
GET    /transactions/{id}
POST   /transactions      body: (see below)
PUT    /transactions/{id} body: (partial, see below)
DELETE /transactions/{id} body: (optional, see below)
```

### POST body fields

| Field | Type | Notes |
|---|---|---|
| `description` | string | Required |
| `date` | string | `YYYY-MM-DD` |
| `amount_cents` | int | Negative = expense, positive = income |
| `paid` | bool | Whether already settled |
| `account_id` | int | Bank account (for regular transactions) |
| `credit_card_id` | int | Credit card (for card transactions) |
| `category_id` | int | |
| `notes` | string | |
| `tags` | array | `[{"name": "tag-name"}]` |
| `recurrence_attributes` | object | `{"periodicity": "monthly"}` — creates a recurring (fixed) series |
| `installments_attributes` | object | `{"periodicity": "monthly", "total": 12}` — creates N installments |

**⚠️ `total_installments` field is ignored by the API.** Use `installments_attributes` instead.

`periodicity` values: `monthly`, `yearly`, `weekly`, `biweekly`, `bimonthly`, `trimonthly`

### PUT additional fields (recurring/installment transactions)

| Field | Type | Notes |
|---|---|---|
| `update_future` | bool | Update this and all future occurrences |
| `update_all` | bool | Update ALL occurrences including past (may affect past balances) |

### DELETE body (recurring/installment transactions)

Same `update_future` / `update_all` flags apply.

### Response fields

`id, description, date, paid, amount_cents, total_installments, installment, recurring, account_id, account_type, category_id, contact_id, notes, attachments_count, credit_card_id, credit_card_invoice_id, paid_credit_card_id, paid_credit_card_invoice_id, oposite_transaction_id, oposite_account_id, created_at, updated_at, tags, attachments, recurrence_id`

- `recurring: true` → fixed recurring transaction
- `total_installments > 1` → part of an installment series
- `oposite_transaction_id` → linked transaction (transfer or credit card payment)

---

## Transfers

Transfers create two linked transaction records — one debit, one credit.  
**Credit cards are NOT accepted** as source or destination.

```
GET    /transfers
GET    /transfers/{id}
POST   /transfers   body: { debit_account_id, credit_account_id, amount_cents, date, paid, tags }
PUT    /transfers/{id}
DELETE /transfers/{id}
```

Response: same fields as a transaction, with `oposite_transaction_id` linking the pair.

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
| 400 | Bad request — usually missing `User-Agent` header |
| 401 | Invalid email or API token |
| 422 | Validation error — check `errors` field in response body |
