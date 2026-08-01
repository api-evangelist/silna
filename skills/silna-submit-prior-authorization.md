---
name: silna-submit-prior-authorization
description: >-
  Submit a prior authorization for a patient plan through the Silna Public API and
  track it to approval, reading back approved treatment codes and the reference number.
api: Silna Public API
source: openapi/silna-openapi.json
method: generated
generated: '2026-07-21'
operations:
- V1PriorAuthorizationResource.post
- V1PriorAuthorizationResource.get
- V1PatientPlanPriorAuthorizationsResource.get
- V1PriorAuthorizationResource.delete
- V1EscalationsResource.get
---

# Submit a Prior Authorization

Request payor approval for a treatment before delivering care.

## Prerequisites
- A Bearer token from the Silna Portal (`Authorization: Bearer <token>`).
- Base URL: `https://app.silnahealth.com/api`.
- An existing patient and patient plan (see the `silna-run-benefits-check` skill).

## Steps

1. **Create the prior authorization.** Call `V1PriorAuthorizationResource.post`
   (`POST /public/v1/prior-authorizations/`) referencing the `patient_plan_id` and the
   requested treatment codes. Send an `Idempotency-Key` header so retries do not create
   duplicates.
2. **Track status.** Poll `V1PriorAuthorizationResource.get`
   (`GET /public/v1/prior-authorizations/{prior_authorization_id}`). An approved
   authorization returns the approved treatment codes, the reference number, and the
   approval letter.
3. **List by plan.** Use `V1PatientPlanPriorAuthorizationsResource.get`
   (`GET /public/v1/patient-plans/{patient_plan_id}/prior-authorizations`) to see all
   authorizations for a plan.
4. **Abort if needed.** Call `V1PriorAuthorizationResource.delete`
   (`DELETE /public/v1/prior-authorizations/{prior_authorization_id}`) to abort — Silna
   archives rather than hard-deletes the record.

## Rules
- Rate limit: 120 requests/minute; honor `Retry-After` and `X-RateLimit-*` on 429.
- Watch for **escalations**: when Silna needs more information it raises one via
  `V1EscalationsResource.get` (`GET /public/v1/escalations/`); these must be resolved in
  the Silna web app.
- Do not use the deprecated `OldV1PriorAuthorizationResource` (`/public/v1/prior_authorizations/...`);
  use the `prior-authorizations` (hyphenated) resource.
