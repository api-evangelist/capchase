---
name: Qualify a buyer and create a financed subscription
description: >-
  Create a buyer company, run Capchase KYB/underwriting qualification, then create a
  subscription that generates a hosted payment link with installment financing terms.
api: openapi/capchase-pay-openapi.yml
operations:
- PayExternalController_createBuyer
- PayExternalController_getBuyer
- PayExternalController_updateBuyerQualification
- PayExternalController_createSubscription
- PayExternalController_getSubscription
---

# Qualify a buyer and create a financed subscription

Use the Capchase Pay v2 API to offer a business buyer installment payment terms at
checkout while the vendor is paid the full contract value upfront.

## Auth & environment
- HTTP Basic auth: `Authorization: Basic base64(clientId:clientSecret)`. Request
  credentials from `pay@capchase.com`. All requests are HTTPS.
- Base URL (production): `https://universe.capchase.com/api/v2`
- Test against Playground first: `https://universe.playground.capchase.com/api/v2`
  (separate credentials).

## Steps
1. **Create the buyer** — `PayExternalController_createBuyer` (`POST /pay/buyers`).
   Send `name`, `legal_name`, `website`, `tax_id`, `country` (ISO-3166-1 alpha-3),
   and address fields. Capchase runs KYB/underwriting behind the scenes.
2. **Check qualification** — `PayExternalController_getBuyer` (`GET /pay/buyers/{id}`).
   Read `kyb_status`, `credit_quality`, and `status`; `kyb_rejected_reasons` explains
   a decline. If qualification is stale, refresh with
   `PayExternalController_updateBuyerQualification` (`POST /pay/buyers/{id}/renew-qualification`).
3. **Create the subscription** — `PayExternalController_createSubscription`
   (`POST /pay/subscriptions`). Send `buyer_id`, `name`, and `items[]` (each with
   `name` + `amount`). The response includes a `payment_link` the buyer completes.
4. **Track status** — `PayExternalController_getSubscription`
   (`GET /pay/subscriptions/{id}`). Watch `finance_status`
   (PAYMENT_LINK_CREATED → … → AGREEMENTS_SIGNED) and `status`
   (DRAFT → READY → BUYER_ACCEPTED → APPROVED → ACTIVE).

## Conventions & errors
- Errors return `{error_code, error_message}` alongside the HTTP status. Handle 401
  (bad/malformed Basic auth), 409 (conflict with current resource status), 422
  (invalid fields), and 429 (rate limit — back off).
- No idempotency-key is supported; guard against duplicate create calls yourself.
- See `conventions/capchase-conventions.yml` and `errors/capchase-error-codes.yml`.
