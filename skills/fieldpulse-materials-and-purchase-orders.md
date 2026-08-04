---
name: Stage FieldPulse materials and raise a purchase order
description: Build a material list for a job, connect items and the job to it, then raise a purchase order against a vendor for the parts.
api: openapi/fieldpulse-api-openapi-original.json
generated: '2026-08-04'
method: generated
source: openapi/fieldpulse-api-openapi-original.json + https://help.fieldpulse.com/api-reference/getting-started
operations:
  - getItems
  - getVendors
  - getVendorsById
  - postMaterialList
  - getMaterialLists
  - postMaterialListsByIdItemConnect
  - postMaterialListsByIdConnect
  - postMaterialListsByIdDisconnect
  - postPurchaseOrders
  - getPurchaseOrdersById
  - putPurchaseOrdersById
---

# Materials and purchase orders

Base URL: `https://ywe3crmpll.execute-api.us-east-2.amazonaws.com/stage`
Auth: `x-api-key: <token>` on every request.

## Shape of this surface

Material lists are the only FieldPulse resource with an explicit graph-editing API: `connect` / `disconnect` / `item-connect` sub-operations rather than embedded arrays. Note the path inconsistency in the published spec — creation and single-record reads use the **singular** `/material-list`, while the collection and the connect operations use the **plural** `/material-lists`. That is what FieldPulse publishes; do not normalise it away.

## Steps

1. **Resolve the parts.** `getItems` — `GET /items`. Item records carry `vendor_id`, `supplier_item_id`, `external_id`, and the QuickBooks account linkage (`qbo_asset_account_id`, `qbo_expense_account_id`, `qbo_purchase_tax_code_id`).

2. **Resolve the vendor.** `getVendors` — `GET /vendors`, then `getVendorsById` — `GET /vendors/{id}` for detail. Vendors are read-only in the public API; they cannot be created or edited over the API.

3. **Create the material list.** `postMaterialList` — `POST /material-list`. The body example carries `item_ids`, `from_entity_id`, and `to_entity_id` — the last two are the polymorphic endpoints of a transfer.

4. **Attach the parts.** `postMaterialListsByIdItemConnect` — `POST /material-lists/{id}/item-connect`. Use this to bind items to the list after creation rather than rewriting the whole record.

5. **Attach the list to work.** `postMaterialListsByIdConnect` — `POST /material-lists/{id}/connect` binds the list to the job or project. `postMaterialListsByIdDisconnect` — `POST /material-lists/{id}/disconnect` reverses it. These are the only two operations that make the list visible on the work record.

6. **Raise the purchase order.** `postPurchaseOrders` — `POST /purchase-orders`. The published body example references `vendor_id`, `customer_id`, `job_id`, `project_id`, `invoice_id`, `assigned_id`, and `pickup_branch_id`. Set only the linkage the account actually uses; a PO can hang off a job, a project, or neither.

7. **Read it back and advance it.** `getPurchaseOrdersById` — `GET /purchase-orders/{id}`, then `putPurchaseOrdersById` — `PUT /purchase-orders/{id}` to update status, quantities, or the order date. FieldPulse made the PO order date optional at creation and stamps it when the PO is emailed or marked ordered, so do not require it up front.

## Idempotency and retries

None of these operations accept an idempotency key. `postMaterialListsByIdItemConnect` and `postMaterialListsByIdConnect` are the risky ones — a blind retry after a timeout can double-attach. Re-read with `getMaterialLists` (using `rel` to expand items) before retrying any connect call.

## Paging and filtering

`page` (from 1), `limit` (default 20, max 100), `calculate_count` for totals, `filter`, `search`, `sort`, and `rel` for relation expansion. See `conventions/fieldpulse-conventions.yml`.

## Errors

`{"message": "..."}` on `400`, `401`, `404`, `500`; `422` and `429` documented but undeclared. On `429`, honour `RateLimit-Reset`. See `errors/fieldpulse-problem-types.yml`.

## No events here

FieldPulse's webhook surface covers Jobs, Estimates, and Invoices only. Material lists, purchase orders, vendors, and items emit **nothing** — reconcile these by polling with `filter` on an updated-at attribute.
