---
name: Onboard a FieldPulse customer and schedule a job
description: Create a customer, attach a contact and a service location, then create and status a job against them using the FieldPulse Open API.
api: openapi/fieldpulse-api-openapi-original.json
generated: '2026-08-04'
method: generated
source: openapi/fieldpulse-api-openapi-original.json + https://help.fieldpulse.com/api-reference/getting-started
operations:
  - getVersion
  - getLeadSource
  - getPipelineStatus
  - postCustomers
  - postCustomersByCustomerIdCustomerContact
  - postLocations
  - getJobsStatusWorkflows
  - getJobsStatusWorkflowStatuses
  - postJobs
  - getJobsByJobId
  - putJobsByJobId
---

# Onboard a customer and schedule a job

Base URL: `https://ywe3crmpll.execute-api.us-east-2.amazonaws.com/stage`

## Before you start

- **Auth.** Every request carries `x-api-key: <token>`. Tokens are issued manually — request one from `support@fieldpulse.com`. There is no OAuth, no scopes, and no self-serve key. Open API access is an Enterprise-plan capability.
- **There is no sandbox.** The documented base URL is production and the docs playground writes to real data. Provision a non-production FieldPulse account before running any step below that is not a `GET`.
- **There is no idempotency key.** If a `POST` times out, do not blind-retry — re-read the collection first (`getCustomers`, `getJobs`) and check whether the record landed, or you will create a duplicate.
- **Rate limit:** 50 requests/second. On `429` read the `RateLimit-Reset` header (epoch seconds) and back off until then. There are no `RateLimit-Remaining` headers on success, so pace conservatively.

## Steps

1. **Confirm connectivity.** `getVersion` — `GET /version`. Cheap, read-only, proves the key works before you write anything.

2. **Read the reference data you will need.**
   - `getLeadSource` — `GET /lead-source`. Returns the lead sources you may set as `lead_source_id` on a customer.
   - `getPipelineStatus` — `GET /pipeline-status`. Returns the pipeline statuses you may set as `pipeline_status_id`.
   Do not guess these ids; they are per-account.

3. **Create the customer.** `postCustomers` — `POST /customers`.
   Body fields published in the spec example include `first_name`, `last_name`, `display_name`, `email`, `phone`, `company_name`, `alt_email`, `notes`, `status`, `has_different_billing_address`, `billing_address_1`, `billing_address_2`, `billing_city`, `billing_state`, `billing_zip_code`, `lead_source_id`, `pipeline_status_id`.
   Keep the returned customer id — every following step references it.

4. **Add a contact.** `postCustomersByCustomerIdCustomerContact` — `POST /customers/{customerId}/customer-contact`. Use this when the billing contact differs from the primary record, or when the account has multiple people.

5. **Add the service location.** `postLocations` — `POST /locations`. Locations are polymorphic: they carry `object_id` (paired with the `object_type` query parameter) plus `primary_customer_contact_id`. Point `object_id` at the customer you just created and set `primary_customer_contact_id` to the contact from step 4 if you created one.

6. **Read the job status workflow.** `getJobsStatusWorkflows` — `GET /jobs/status-workflows`, then `getJobsStatusWorkflowStatuses` — `GET /jobs/status-workflow-statuses`. Jobs carry `status_id` and `status_workflow_id`; both are account-defined. Resolve them here rather than hardcoding.

7. **Create the job.** `postJobs` — `POST /jobs`. Reference the `customer_id` from step 3, the `location_id` from step 5, and the `status_workflow_id` / `status_id` from step 6. Set the schedule fields on the same call.

8. **Verify.** `getJobsByJobId` — `GET /jobs/{jobId}`. Use the `rel` query parameter to expand related records in one read instead of issuing follow-up calls.

9. **Advance the job.** `putJobsByJobId` — `PUT /jobs/{jobId}` to change `status_id`, reschedule (`start` / `end`), or reassign. FieldPulse uses `PUT`, not `PATCH`.

## Reading collections

Every list endpoint (`getCustomers`, `getJobs`, …) shares the same conventions:

- `page` — starts at 1, defaults to 1.
- `limit` — defaults to 20, maximum 100.
- `calculate_count` — set `true` to get a total count; totals are **opt-in** and absent by default.
- `filter` — array of attribute / operator / value parts.
- `search` — free-text.
- `sort` — array of attribute and order values. Some endpoints instead take `sort_by` + `sort_dir` (`asc`/`desc`); check the operation.
- `rel` — array of relation names to expand inline.
- `asn` — filter by assignment (`team_id`, `assigned_members`).

## Errors

Responses are `application/json` with a bare `{"message": "..."}` — there is no error code and no `application/problem+json`.

| Status | Meaning | What to do |
|---|---|---|
| 400 | Bad Request | A required body field is missing or wrong. Re-read the request-body description: `optional` = not required, `nullable` = may be null. |
| 401 | Unauthorized | `x-api-key` missing or invalid. Do not retry. |
| 404 | Not Found | Unknown resource id, or an unrecognized request URL. |
| 422 | Unprocessable Entity | Authentication validation error. Documented in Getting Started but **not** declared on any operation, so a generated client will not model it. |
| 429 | Quota exceeded / Throttled | Two distinct conditions — a monthly quota and the 50 rps throttle. Read `RateLimit-Reset` and wait. Also undeclared in the spec. |
| 500 | Internal Server Error | Retryable, but see the idempotency warning above — never blind-retry a write. |

## Events

If you need to react to job progress rather than poll, FieldPulse emits webhooks for `Job Created`, `Job Custom Status Update`, `Job Workflow Custom Status Update`, `Job Start Time Update`, and `Job End Time Update`. Setup is **not** self-serve — FieldPulse Support must configure it — and deliveries are **not signed**, so verify by source IP or by re-reading the record with `getJobsByJobId` before acting on a payload. See `asyncapi/fieldpulse-events-webhooks.yml`.
