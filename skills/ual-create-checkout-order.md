---
name: Create a Ualá Bis checkout order and track it to approval
description: >-
  Mint an access token, create a hosted checkout order, hand the buyer the checkout link, and
  track the order to APPROVED via webhook or polling.
api: openapi/ual-bis-cobros-online-v2-openapi.yml
operations: [createToken, createOrder, getOrder, listOrders]
generated: '2026-07-21'
method: generated
---

# Create a Ualá Bis checkout order and track it

1. **Authenticate** — `createToken`: POST `/auth/token` on `https://auth.developers.ar.ua.la/v2/api`
   (test: `auth.stage.developers.ar.ua.la`) with JSON body `username`, `client_id`,
   `client_secret_id`, `grant_type: client_credentials`. Use the returned token as
   `Authorization: Bearer <token>` on every call. Credentials come from the Ualá Bis welcome
   email or app; never hard-code them.
2. **Create the order** — `createOrder`: POST `/checkout` on
   `https://checkout.developers.ar.ua.la/v2/api` with `amount` (string, 25.00–9999999.00,
   single decimal point), `description`, `callback_success`, `callback_fail`; optionally
   `notification_url` (webhook) and `external_reference` (your order ID).
3. **Redirect the buyer** to `links.checkout_link` from the response. The order starts as
   `PENDING`.
4. **Track status** — prefer the webhook: Ualá POSTs `{uuid, external_reference, status,
   created_date, api_version}` to your `notification_url` on `APPROVED`, `PROCESSED`, or
   `REJECTED`; respond HTTP 200 or delivery is retried up to 3 more times. Otherwise poll
   `getOrder` (`GET /orders/{uuid}`) or reconcile in bulk with `listOrders`
   (`GET /orders?limit=&fromDate=&toDate=&status=`, cursor via `last_search_key`,
   `limit` < 50, default 10).
5. **Interpret status**: `APPROVED` = paid and disbursed to the Ualá account (only then are
   `taxes`/`commissions` populated); `PROCESSED` = paid, disbursement pending; `REJECTED` =
   payment failed.
6. **Handle errors** by the envelope `{code, message, errors[]}` — `request_error` = fix the
   payload (see errors/ual-problem-types.yml); `Unauthorized`/explicit-deny = mint a fresh
   token; `api_error` = retry later. There is no idempotency key — deduplicate on your
   `external_reference` before creating a second order.
