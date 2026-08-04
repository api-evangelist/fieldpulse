---
name: Fieldpulse
description: Use when managing field service operations, scheduling jobs and technicians, creating estimates and invoices, tracking payments, managing customers, building workflows, integrating with accounting software, or working with the FieldPulse API to automate business processes.
metadata:
    mintlify-proj: fieldpulse
    version: "1.0"
---

# FieldPulse Skill Guide

## Product Summary

FieldPulse is a field service management platform for scheduling jobs, managing customers, creating estimates and invoices, collecting payments, and tracking team performance. Agents use it to help users set up workflows, configure integrations (QuickBooks, Xero, Zapier), manage mobile field operations, build custom reports, and automate business processes via the REST API.

**Key files and paths:**
- Web app: https://webapp.fieldpulse.com
- Mobile app: iOS (App Store) and Android (Google Play)
- API base URL: https://ywe3crmpll.execute-api.us-east-2.amazonaws.com/stage
- API authentication: x-api-key header (request from support@fieldpulse.com)
- Primary docs: https://help.fieldpulse.com

**Core concepts:** Customers, Jobs, Estimates, Invoices, Payments, Workflows, Teams, Scheduling, Reporting, Custom Fields, Integrations.

---

## When to Use

Reach for this skill when:

- **Setting up FieldPulse:** User is configuring account, adding users/teams, setting up company settings, or establishing workflows
- **Managing work:** Creating jobs, scheduling technicians, assigning teams, tracking job status, managing site visits
- **Billing:** Creating estimates, converting to invoices, collecting payments, managing payment methods, handling refunds
- **Customer management:** Creating/importing customers, managing contacts and locations, tracking lead sources, organizing with tags
- **Reporting:** Building custom reports, filtering data, exporting for analysis, creating dashboards
- **Mobile operations:** Helping field technicians use the mobile app, clocking in/out, updating job status, capturing photos/signatures
- **Integrations:** Connecting QuickBooks, Xero, Zapier, or other third-party tools; syncing data
- **API work:** Building custom integrations, automating workflows, pushing/pulling data from external systems
- **Troubleshooting:** Connectivity issues, permission problems, data import errors, sync failures

---

## Quick Reference

### Core Workflows

| Task | Web Path | Mobile Path |
|------|----------|------------|
| Create customer | Customers > Create Customer | Customers tab > Create New |
| Create job | Jobs > Create Job | Schedule/Work tab > Create New |
| Create estimate | Estimates > Create Estimate | Sales tab > Estimates > Create New |
| Create invoice | Invoices > Create Invoice | Sales tab > Invoices > Create New |
| Collect payment | Job/Invoice > Collect Payment | Sales tab > Invoices > Collect Payment |
| View schedule | Schedule tab | Schedule tab (List/Calendar/Map/Dispatch view) |
| Clock in/out | Timesheets tab | Timesheets tab or Dashboard |
| Generate report | Reporting > [Record Type] > Raw Data | Not available on mobile |

### API Endpoints (Common)

| Resource | Methods | Use Case |
|----------|---------|----------|
| /customers | GET, POST, PUT, DELETE | Create/update customer records |
| /jobs | GET, POST, PUT, DELETE | Create/update jobs and assign technicians |
| /estimates | GET, POST, PUT, DELETE | Build estimates, convert to invoices |
| /invoices | GET, POST, PUT, DELETE | Create invoices, update status |
| /payments | GET, POST, PUT, DELETE | Record payments, process refunds |
| /custom-fields | GET, POST, PUT, DELETE | Create custom fields on records |
| /material-lists | GET, POST, PUT, DELETE | Create material lists for jobs |
| /subtasks | GET, POST, PUT, DELETE | Add subtasks to jobs |

### Key Settings Locations

- **Company Settings:** Left menu > Company Settings
  - Users & Teams
  - Customers (import, lead sources, tags)
  - Jobs (numbering, custom status workflows)
  - Estimates & Invoices (defaults, templates, tax)
  - Payments (payment providers, fee recovery)
  - Communications (notification triggers, templates)

- **User Roles:** Admin, Manager, Dispatcher, Service Agent, Limited Agent (permissions control what users see)

---

## Decision Guidance

### When to Use Web vs. Mobile

| Scenario | Use Web | Use Mobile |
|----------|---------|-----------|
| Creating customers in bulk | ✓ (import template) | ✗ |
| Updating job status in field | ✗ | ✓ (real-time) |
| Building estimates/invoices | ✓ (full editor) | ✓ (simplified) |
| Generating reports | ✓ (full reporting) | ✗ |
| Capturing photos/signatures | ✗ | ✓ (camera access) |
| Scheduling jobs | ✓ (map/calendar view) | ✓ (limited) |
| Collecting payments | ✓ | ✓ (Tap to Pay on iPhone) |

### When to Use API vs. UI

| Scenario | Use API | Use UI |
|----------|---------|--------|
| One-time data import | ✗ | ✓ (import template) |
| Ongoing data sync with external system | ✓ | ✗ |
| Bulk customer/job creation | ✓ (POST loop) | ✓ (import) |
| Real-time job status updates | ✓ (webhooks) | ✓ (manual) |
| Custom reporting/analytics | ✓ (export data) | ✓ (Raw Data Reporting) |
| Automating lead capture from website | ✓ | ✗ |

### Estimate vs. Invoice

| Aspect | Estimate | Invoice |
|--------|----------|---------|
| Purpose | Get customer approval before work | Bill customer after work |
| Conversion | Convert to invoice once accepted | Created from estimate or standalone |
| Signature | Can require customer signature | Not required |
| Payment | Not expected | Expected (can set due date) |
| Status | Draft → Sent → Accepted → Converted | Draft → Sent → Paid/Unpaid |

---

## Workflow

### Typical Job-to-Payment Cycle

1. **Create Customer**
   - Navigate to Customers > Create Customer
   - Fill: Status, Account Type, Primary Contact, Address, Tags
   - Save and note the Customer ID (visible in URL)

2. **Create Job**
   - Go to Jobs > Create Job
   - Link customer, add title/subtitle, assign team members
   - Set date/time and use "Find Availability" to check schedules
   - Add job tags, custom status workflow, notes
   - Save (team gets notified automatically)

3. **Create Estimate (if needed)**
   - From job record, click Actions > Create Estimate
   - Add line items (materials, labor) from item list or create new
   - Set expiration date, add notes, attach files
   - Save as Draft, then View Estimate to send to customer
   - Customer accepts (status auto-updates to Accepted)

4. **Convert to Invoice**
   - Once estimate accepted, click Convert to Invoice
   - Review line items, adjust if needed
   - Add payment terms, set due date
   - Save and send to customer

5. **Collect Payment**
   - On invoice, click Collect Payment
   - Choose payment method (FieldPulse Payments, manual entry, etc.)
   - Process payment (on-site or email link)
   - Payment status updates automatically

6. **Report & Analyze**
   - Go to Reporting > Invoices > Raw Data
   - Filter by date range, customer, status
   - Group by team member, customer, or date
   - Export CSV for accounting or analysis

### Data Import Workflow

1. Download import template from Company Settings > [Record Type] > Import
2. Fill template in Excel or Google Sheets (do not alter structure)
3. Click Verify to flag errors (fields in red are invalid)
4. Fix errors and re-verify until all records show Valid
5. Export verified data as .xlsx
6. Upload back into FieldPulse using Upload button
7. Confirm import completed and spot-check records

### API Integration Workflow

1. Request API key from support@fieldpulse.com
2. Review API documentation at https://documenter.getpostman.com/view/35988189/2sA3XLEjFd
3. Identify endpoints needed (e.g., /customers, /jobs, /invoices)
4. Build request with x-api-key header and JSON body
5. Test with GET requests first (retrieve data)
6. Move to POST/PUT for create/update operations
7. Implement error handling (429 = rate limit, 401 = auth failure)
8. Monitor rate limit: 50 requests/second max

---

## Common Gotchas

- **Email domain must be lowercase** when entering customer contact emails, or email sends will fail
- **Job subtitle limited to 150 characters** — longer text will be truncated
- **Import template structure cannot be altered** — adding/removing columns breaks the import
- **API rate limit is 50 requests/second** — batch requests or implement backoff logic
- **Webhooks only available for job status changes** — other record types require polling
- **Custom fields must be created in Company Settings first** before they appear on records
- **Archived jobs are hidden by default** — use Search & Filter to find them
- **Deleted jobs cannot be permanently removed** — archive or move to Canceled status instead
- **Tax rates apply only to items marked as taxable** — verify item settings before invoicing
- **Mobile app permissions are role-based** — Service Agents see limited features compared to Admins
- **Offline mode has limited functionality** — estimates/invoices/timesheets only; no reporting
- **Duplicate customers can cause sync issues** — use merge tool or import verification to prevent
- **Payment methods must be saved before collecting** — "Save Payment Method on File" is a separate step
- **Estimate expiration date is set per-estimate** — default from Company Settings can be overridden
- **Job templates save structure but not customer/date** — reuse for similar work types, not one-off jobs

---

## Verification Checklist

Before submitting work, confirm:

- [ ] **Customers:** All required fields filled (name, email, phone, address); no duplicate records
- [ ] **Jobs:** Assigned to correct team members; date/time set; status workflow selected; no scheduling conflicts
- [ ] **Estimates:** Line items added; tax rates correct; expiration date set; customer notified
- [ ] **Invoices:** Converted from estimate or created standalone; payment terms set; due date visible
- [ ] **Payments:** Recorded with correct amount, date, and method; status shows Paid/Pending as appropriate
- [ ] **Imports:** Verified before upload; all records show Valid status; spot-checked 3-5 records post-import
- [ ] **API calls:** Response status is 200/201/202; JSON payload matches expected schema; rate limit not exceeded
- [ ] **Workflows:** Custom status workflow assigned to jobs; statuses match business process; team understands flow
- [ ] **Reports:** Date range correct; filters applied; columns selected; exported CSV opens without errors
- [ ] **Mobile:** User can log in; permissions allow access to assigned jobs; offline mode syncs after reconnect
- [ ] **Integrations:** Accounting sync completed; no duplicate records in QuickBooks/Xero; payment status matches

---

## Resources

**Comprehensive navigation:** https://help.fieldpulse.com/llms.txt

**Critical documentation:**
1. [Getting Started Guide](https://help.fieldpulse.com/getting-started) — Account setup, first tasks, role-based guides
2. [Using FieldPulse](https://help.fieldpulse.com/using-fieldpulse) — Jobs, customers, estimates, invoices, payments, reporting
3. [API Reference](https://help.fieldpulse.com/api-reference/overview) — Endpoints, authentication, rate limits, webhooks

**Additional resources:**
- [FieldPulse360 Training Archives](https://help.fieldpulse.com/training-resources/fieldpulse360/fieldpulse360-archives) — Video walkthroughs and best practices
- [Integrations & Partners](https://help.fieldpulse.com/integrations-partners) — QuickBooks, Xero, Zapier, supplier integrations
- [Features & Add-Ons](https://help.fieldpulse.com/features-add-ons) — ClearPath workflows, Custom Forms, Engage phone system, Sales Suite
- [Troubleshooting & Technical Specs](https://help.fieldpulse.com/troubleshooting-technical-specs) — Device compatibility, character limits, common fixes

**Support:** support@fieldpulse.com or chat in bottom right of help center

---

> For additional documentation and navigation, see: https://help.fieldpulse.com/llms.txt