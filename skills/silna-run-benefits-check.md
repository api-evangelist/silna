---
name: silna-run-benefits-check
description: >-
  Run an on-demand insurance benefits check for a patient's plan with the Silna
  Public API and read back coverage details (active status, copay, deductible).
api: Silna Public API
source: openapi/silna-openapi.json
method: generated
generated: '2026-07-21'
operations:
- V1PatientsResource.post
- V1PatientPlanResource.post
- V1BenefitsCheckResource.post
- V1BenefitsCheckResource.get
- V1BenefitsCheckReportResource.post
---

# Run a Benefits Check

Confirm a patient's insurance coverage before delivering care.

## Prerequisites
- A Bearer token created in the Silna Portal (https://app.silnahealth.com/), scoped
  to your provider or MSO. Send it as `Authorization: Bearer <token>` on every request.
- Base URL: `https://app.silnahealth.com/api`.

## Steps

1. **Ensure the patient exists.** Create with `V1PatientsResource.post`
   (`POST /public/v1/patients/`), or upsert idempotently with
   `V1PatientsResource.put` (`PUT /public/v1/patients/`) using your own `source_id`.
2. **Ensure the patient plan exists.** Create with `V1PatientPlanResource.post`
   (`POST /public/v1/patient-plans/`). Patient Plans are immutable — to change one,
   archive it and create a new plan.
3. **Create the benefits check.** Call `V1BenefitsCheckResource.post`
   (`POST /public/v1/benefits-checks/`) referencing the `patient_plan_id`. Pass an
   `Idempotency-Key` header to make the create safe to retry.
4. **Poll for the result.** Call `V1BenefitsCheckResource.get`
   (`GET /public/v1/benefits-checks/{benefits_check_id}`) until it resolves. The result
   reports whether the plan is active plus copay, deductible, and related coverage.
5. **(Optional) Generate a report.** Call `V1BenefitsCheckReportResource.post`
   (`POST /public/v1/benefits-checks/{benefits_check_id}/reports`) for a shareable report.

## Rules
- Rate limit: 120 requests/minute. Honor `Retry-After` and `X-RateLimit-*` headers on 429.
- Errors return `{ "message": ... }` (401 also returns `type`). Idempotency keys are
  honored only for 2XX responses and expire after 30 days.
- If Silna needs more information it raises an **escalation** — resolve it in the Silna
  web app; check `V1EscalationsResource.get` for open escalations.
