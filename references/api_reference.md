# Organizze API Reference

Base URL: `https://api.organizze.com.br/rest/v2`  
Auth: HTTP Basic (`email:api_token`). Header `User-Agent` **required** (400 without it).  
API key: https://app.organizze.com.br/configuracoes/api-keys

## Table of Contents
1. [Pagination & Filters](#pagination--filters)
2. [Users](#users)
3. [Accounts](#accounts)
4. [Categories](#categories)
5. [Credit Cards](#credit-cards)
6. [Invoices](#invoices)
7. [Transactions](#transactions)
8. [Transfers](#transfers)
9. [Budgets](#budgets)
10. [Errors](#errors)

---

## Pagination & Filters

Transactions and invoices default to **current month/year** without a period.  
Use `?start_date=YYYY-MM-DD&end_date=YYYY-MM-DD`. Dates are rounded to full months.  
Filter transactions by account: `?account_id={id}`.

---

## Users

```
GET /users/{id}
```

```json
{ "id": 3, "name": "Esdras Mayrink", "email": "email@example.com", "role": "admin" }
```

---

## Accounts

```
GET    /accounts
GET    /accounts/{id}
POST   /accounts        body: { name, type, description, default }
PUT    /accounts/{id}
DELETE /accounts/{id}
```

`type` values: `checking`, `savings`, `other`

```json
{
  "id": 3,
  "name": "Bradesco CC",
  "description": "Conta corrente",
  "archived": false,
  "created_at": "2015-06-22T16:17:03-03:00",
  "updated_at": "2015-08-31T22:24:24-03:00",
  "default": true,
  "type": "checking"
}
```

---

## Categories

```
GET    /categories
GET    /categories/{id}
POST   /categories      body: { name }
PUT    /categories/{id} body: { name }
DELETE /categories/{id} body: { replacement_id }
```

`replacement_id` in DELETE: reassigns all transactions to that category before deleting.

```json
{ "id": 1, "name": "Lazer", "color": "438b83", "parent_id": null }
```

`parent_id: null` = top-level. Sub-categories have a parent.

---

## Credit Cards

```
GET    /credit_cards
GET    /credit_cards/{id}
POST   /credit_cards      body: { name, card_network, due_day, closing_day, limit_cents }
PUT    /credit_cards/{id} body: { name, due_day, closing_day, update_invoices_since }
DELETE /credit_cards/{id}
```

```json
{
  "id": 3,
  "name": "Visa Exclusive",
  "description": null,
  "card_network": "visa",
  "closing_day": 4,
  "due_day": 17,
  "limit_cents": 1200000,
  "kind": "credit_card",
  "archived": false,
  "default": false,
  "created_at": "2015-06-22T16:45:30-03:00",
  "updated_at": "2015-09-01T18:18:48-03:00"
}
```

---

## Invoices

```
GET /credit_cards/{id}/invoices
GET /credit_cards/{id}/invoices/{invoice_id}
GET /credit_cards/{id}/invoices/{invoice_id}/payments
```

### Invoice list item

```json
{
  "id": 186,
  "date": "2015-07-17",
  "starting_date": "2015-06-03",
  "closing_date": "2015-07-04",
  "amount_cents": 30000,
  "payment_amount_cents": -70000,
  "balance_cents": 100000,
  "previous_balance_cents": 0,
  "credit_card_id": 3
}
```

- `date`: due date
- `balance_cents`: remaining amount due (`!= 0` means unpaid balance)
- `previous_balance_cents`: balance carried over from previous invoice

### Detailed invoice (includes transactions and payments)

```json
{
  "id": 186,
  "date": "2015-07-17",
  "starting_date": "2015-06-03",
  "closing_date": "2015-07-04",
  "amount_cents": 30000,
  "payment_amount_cents": -70000,
  "balance_cents": 100000,
  "previous_balance_cents": 0,
  "credit_card_id": 3,
  "transactions": [
    {
      "id": 19,
      "description": "Gasto no cartão",
      "date": "2015-06-03",
      "paid": true,
      "amount_cents": -5000,
      "total_installments": 1,
      "installment": 1,
      "recurring": false,
      "account_id": 3,
      "account_type": "CreditCard",
      "category_id": 21,
      "contact_id": null,
      "notes": "",
      "attachments_count": 0
    },
    {
      "id": 12,
      "description": "SAQUE LOT",
      "date": "2015-06-06",
      "paid": true,
      "amount_cents": -15000,
      "total_installments": 5,
      "installment": 1,
      "recurring": false,
      "account_id": 3,
      "account_type": "CreditCard",
      "category_id": 21,
      "contact_id": null,
      "notes": "",
      "attachments_count": 0
    }
  ],
  "payments": [
    {
      "id": 83,
      "description": "Pagamento Julho de 2015",
      "date": "2015-09-01",
      "paid": true,
      "amount_cents": -20000,
      "account_id": 3,
      "account_type": "Account",
      "category_id": 21
    }
  ]
}
```

---

## Transactions

```
GET    /transactions?start_date=YYYY-MM-DD&end_date=YYYY-MM-DD[&account_id={id}]
GET    /transactions/{id}
POST   /transactions
PUT    /transactions/{id}
DELETE /transactions/{id}
```

### POST body fields

| Field | Type | Notes |
|---|---|---|
| `description` | string | Required |
| `date` | string | `YYYY-MM-DD` |
| `amount_cents` | int | Negative = expense, positive = income |
| `paid` | bool | Whether already settled |
| `account_id` | int | Bank account (regular transactions) |
| `credit_card_id` | int | Credit card (card transactions) |
| `category_id` | int | |
| `notes` | string | |
| `tags` | array | `[{"name": "tag-name"}]` |
| `recurrence_attributes` | object | `{"periodicity": "monthly"}` — fixed recurring series |
| `installments_attributes` | object | `{"periodicity": "monthly", "total": 12}` — N installments |

**⚠️ `total_installments` is ignored by the API.** Use `installments_attributes`.

`periodicity` values: `monthly`, `yearly`, `weekly`, `biweekly`, `bimonthly`, `trimonthly`

### PUT/DELETE extra fields (recurring or installment transactions)

| Field | Notes |
|---|---|
| `update_future: true` | Apply to this + all future occurrences |
| `update_all: true` | Apply to ALL occurrences including past (may affect settled balances) |

### Full transaction response

```json
{
  "id": 31,
  "description": "Lanche",
  "date": "2015-09-02",
  "paid": false,
  "amount_cents": -2098,
  "total_installments": 1,
  "installment": 1,
  "recurring": false,
  "account_id": 3,
  "account_type": "Account",
  "category_id": 18,
  "contact_id": null,
  "notes": "",
  "attachments_count": 0,
  "credit_card_id": null,
  "credit_card_invoice_id": null,
  "paid_credit_card_id": null,
  "paid_credit_card_invoice_id": null,
  "oposite_transaction_id": null,
  "oposite_account_id": null,
  "created_at": "2015-08-20T18:00:20-03:00",
  "updated_at": "2015-09-01T18:14:54-03:00",
  "tags": [],
  "attachments": [],
  "recurrence_id": null
}
```

### Key fields explained

| Field | Notes |
|---|---|
| `account_type` | `"Account"` = bank, `"CreditCard"` = credit card |
| `recurring` | `true` when part of a fixed recurring series |
| `total_installments` | Total installments (e.g. 12); `installment` = current number |
| `recurrence_id` | Links all occurrences of a recurring/installment series |
| `oposite_transaction_id` | Linked record (transfer pair or credit card payment) |
| `oposite_account_id` | Account of the linked record |
| `paid_credit_card_id` | Set when this transaction is a credit card invoice payment |
| `credit_card_invoice_id` | Invoice this transaction belongs to |

### Installment transaction example

```json
{
  "id": 97,
  "description": "Despesa parcelada",
  "date": "2015-09-16",
  "paid": false,
  "amount_cents": 0,
  "total_installments": 12,
  "installment": 1,
  "recurring": false,
  "recurrence_id": 42,
  "account_id": 3,
  "category_id": 21
}
```

### Recurring (fixed) transaction example

```json
{
  "id": 97,
  "description": "Despesa fixa",
  "date": "2015-09-16",
  "paid": false,
  "amount_cents": 0,
  "total_installments": 1,
  "installment": 1,
  "recurring": true,
  "recurrence_id": 43,
  "account_id": 3,
  "category_id": 21
}
```

---

## Transfers

Creates two linked records — one debit, one credit. **Credit cards not accepted.**

```
GET    /transfers
GET    /transfers/{id}
POST   /transfers   body: { debit_account_id, credit_account_id, amount_cents, date, paid, tags }
PUT    /transfers/{id}
DELETE /transfers/{id}
```

### POST body

```json
{
  "debit_account_id": 3,
  "credit_account_id": 4,
  "amount_cents": 10000,
  "date": "2015-09-01",
  "paid": true,
  "tags": [{"name": "ajuste"}]
}
```

### Response (debit side)

```json
{
  "id": 10,
  "description": "Transferência",
  "date": "2015-09-01",
  "paid": true,
  "amount_cents": -10000,
  "total_installments": 1,
  "installment": 1,
  "recurring": false,
  "account_id": 3,
  "category_id": 21,
  "notes": null,
  "attachments_count": 0,
  "credit_card_id": null,
  "credit_card_invoice_id": null,
  "paid_credit_card_id": null,
  "paid_credit_card_invoice_id": null,
  "oposite_transaction_id": 11,
  "oposite_account_id": 4,
  "created_at": "2015-09-01T23:42:29-03:00",
  "updated_at": "2015-09-01T23:42:29-03:00",
  "tags": [{"name": "ajuste"}],
  "attachments": [],
  "recurrence_id": null
}
```

`oposite_transaction_id: 11` is the credit side (positive amount in account 4).

---

## Budgets (Metas)

```
GET /budgets                    # current month
GET /budgets/{year}             # full year
GET /budgets/{year}/{month}     # specific month
```

```json
[
  {
    "amount_in_cents": 150000,
    "category_id": 17,
    "date": "2018-08-01",
    "activity_type": 0,
    "total": 0,
    "predicted_total": 0,
    "percentage": "0.0"
  }
]
```

- `amount_in_cents`: budget limit for the category
- `total`: actual spending so far
- `predicted_total`: projected spending
- `percentage`: spent / budget ratio

---

## Errors

| Status | Meaning |
|---|---|
| 400 | Bad request — usually missing `User-Agent` header |
| 401 | Invalid email or API token — `{"error": "Não autorizado"}` |
| 422 | Validation error — check `errors` field in response body |
