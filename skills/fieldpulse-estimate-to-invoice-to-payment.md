---
name: Take a FieldPulse estimate through to invoice and payment
description: Build an estimate for a customer, convert the work into an invoice, apply a price update, and record a payment against it.
api: openapi/fieldpulse-api-openapi-original.json
generated: '2026-08-04'
method: generated
source: openapi/fieldpulse-api-openapi-original.json + https://help.fieldpulse.com/api-reference/getting-started
operations:
  - getCustomers
  - getItems
  - postEstimates
  - getEstimatesByEstimateId
  - getEstimatesQbClasses
  - postInvoices
  - getInvoicesByInvoiceId
  - putInvoicesByInvoiceIdPriceUpdate
  - postPayments
  - getPayments
---

# Estimate → invoice → payment

Base URL: `https://ywe3crmpll.execute-api.us-east-2.amazonaws.com/stage`
Auth: `x-api-key: <token>` on every request.

## Warnings before you write

- **Money operations have no idempotency key.** `postInvoices` and especially `postPayments` are not idempotent. A retried `postPayments` records a **second payment**. On any timeout or `5xx`, call `getPayments` and filter by `invoice_id` before retrying.
- **No sandbox.** These operations move real money records in a real account.
- **No response schemas.** The OpenAPI declares `application/json` with no schema on every `200`, so field names in responses are not machine-readable from the contract. Read one live response and pin your parser to it, and re-check after any FieldPulse release — there is no API changelog to warn you.

## Steps

1. **Resolve the customer.** `getCustomers` — `GET /customers` with `search` or `filter` to find the account, or use a `customer_id` you already hold.

2. **Resolve line items.** `getItems` — `GET /items`. Items carry supplier and QuickBooks linkage (`vendor_id`, `supplier_item_id`, `qbo_id`, `qbo_income_account_id`, `qbo_sales_tax_code_id`). Use real item ids rather than free-text lines when the account maintains a pricebook.

3. **Optional — resolve QuickBooks classes.** `getEstimatesQbClasses` — `GET /estimates/qb-classes`. Only relevant when the account syncs to QuickBooks Online and classes are required on the estimate.

4. **Create the estimate.** `postEstimates` — `POST /estimates`. Reference the `customer_id`. Estimates carry `title`, `status_id`, `status_workflow_id`, `author_id`, and the totals (`total`, `subtotal`, `tax`, `discount`) that also appear on the `Estimate Created` webhook payload.

5. **Read it back.** `getEstimatesByEstimateId` — `GET /estimates/{estimateId}`. Use `rel` to expand line items and the customer in one read. `rel[public_link]` and `rel[public_links]` expand the customer-facing estimate link.

6. **Create the invoice.** `postInvoices` — `POST /invoices`. There is no "convert estimate to invoice" operation in the public API; you create the invoice against the same `customer_id` and carry the agreed lines across yourself.

7. **Apply a price update if needed.** `putInvoicesByInvoiceIdPriceUpdate` — `PUT /invoices/{invoiceId}/price-update`. This is the dedicated repricing operation; use it rather than a full `PUT /invoices/{invoiceId}` when only pricing changes.

8. **Record the payment.** `postPayments` — `POST /payments`. Payments reference `invoice_id` and `payment_method_id`, and may carry `square_transaction_id` for card transactions settled through Square. Capture the returned payment id.

9. **Reconcile.** `getInvoicesByInvoiceId` — `GET /invoices/{invoiceId}` and `getPayments` — `GET /payments` filtered to the invoice. Confirm the invoice status advanced before reporting success. Do not infer success from the `postPayments` HTTP status alone.

## Rate limits and paging

50 requests/second; `429` returns `RateLimit-Reset` (epoch). A separate monthly quota also returns `429` — the two are distinguished only by the `message` string. Collections take `page` (from 1), `limit` (default 20, max 100), `calculate_count`, `filter`, `sort`, and `rel`.

## Events

FieldPulse emits `Invoice Created`, `Invoice Custom Status Update`, `Invoice Workflow Custom Status Update`, `Estimate Created`, `Estimate Custom Status Update`, and `Estimate Workflow Custom Status Update`. The status-update payloads use the change envelope — `{"old_value": …, "new_value": …, "object": {…}}` — so a status transition arrives with both sides. Webhooks are unsigned and must be configured by FieldPulse Support. See `asyncapi/fieldpulse-events-webhooks.yml`.

## Errors

`{"message": "..."}` on `400`, `401`, `404`, `500`. `422` and `429` are documented in Getting Started but declared on **no** operation, so handle them defensively rather than trusting a generated client. Full catalogue: `errors/fieldpulse-problem-types.yml`.
