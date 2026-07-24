---
name: Reconcile subscriptions and pull instalment receipts
description: >-
  List a vendor's Capchase Pay subscriptions, inspect their status, and download the
  signed receipt PDF for a returned instalment for accounting reconciliation.
api: openapi/capchase-pay-openapi.yml
operations:
- PayExternalController_listSubscriptions
- PayExternalController_getSubscription
- PayExternalController_getInstalmentReceipt
- PayExternalController_delete
---

# Reconcile subscriptions and pull instalment receipts

Use the Capchase Pay v2 API to reconcile financed deals and retrieve receipts.

## Auth & environment
- HTTP Basic auth: `Authorization: Basic base64(clientId:clientSecret)` over HTTPS.
- Base URL (production): `https://universe.capchase.com/api/v2`.

## Steps
1. **List subscriptions** — `PayExternalController_listSubscriptions`
   (`GET /pay/subscriptions`). Optional filters: `buyer_id`, plus `offset` (default 0)
   and `limit` (default 100) for pagination. Page until fewer than `limit` are returned.
2. **Inspect a subscription** — `PayExternalController_getSubscription`
   (`GET /pay/subscriptions/{id}`). Read `status`, `amount`, `amount_net`,
   `amount_fee`, and the `breakdown` of installments.
3. **Download an instalment receipt** — `PayExternalController_getInstalmentReceipt`
   (`GET /pay/subscriptions/{key}/instalment/{instalmentKey}/receipt`). Returns a
   signed `url` to the receipt PDF for a returned instalment.
4. **Archive if needed** — `PayExternalController_delete`
   (`DELETE /pay/subscriptions/{key}`). Only allowed for DRAFT, READY,
   BUYER_ACCEPTED, APPROVED, ACTIVE, or ERROR subscriptions; cannot be undone.

## Conventions & errors
- Pagination is offset/limit. Errors return `{error_code, error_message}`; handle 404
  (unknown id), 409 (archive not allowed in current status), and 429 (rate limit).
- See `conventions/capchase-conventions.yml` and `data-model/capchase-data-model.yml`.
