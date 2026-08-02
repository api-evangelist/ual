---
name: Refund a Ualá Bis order and confirm the result
description: >-
  Verify an order is refundable, create the refund, and confirm the final result via the
  refund webhook.
api: openapi/ual-bis-cobros-online-v2-openapi.yml
operations: [createToken, getOrder, refundOrder]
generated: '2026-07-21'
method: generated
---

# Refund a Ualá Bis order

1. **Authenticate** — `createToken` (see the checkout skill) and use the Bearer token.
2. **Pre-check the order** — `getOrder` (`GET /orders/{uuid}`): the refund will only be
   accepted if status is `APPROVED`, there is no refund in progress, the order is less than
   90 days old, and it was created on API Checkout v2.
3. **Create the refund** — `refundOrder`: POST `/orders/{uuid}/refund` with `amount` (string)
   and optionally `notification_url`. A 200 response returns `{"status": "INITIATED"}` — this
   is not the final result.
4. **Confirm the result** via the refund webhook on your `notification_url`: `REFUNDED`
   (money returned per the buyer's bank timing) or `NOT_REFUNDED` (a validation failed).
   Respond HTTP 200 to acknowledge. Without a webhook, re-poll `getOrder` for status
   `REFUNDED`.
5. **Handle rejections**: 400 `The order is not valid to refund` lists the failed
   pre-condition; 404 `No record found` means a bad uuid; 403 `You are not allowed to perform
   this action` means the credentials lack refund permission. Full catalog:
   errors/ual-problem-types.yml.
