---
name: Manage Erply webhooks
description: Register, list and remove Erply Webhook Manager configurations to receive real-time POS and sales-order events instead of polling.
api: openapi/erply-webhook-openapi.json
operations:
  - getWebhookConfigurations
  - addWebhookConfiguration
  - updateWebhookConfiguration
  - removeWebhookConfiguration
  - getEvents
---

# Manage Erply webhooks

Use this skill to subscribe to Erply events via the Webhook Manager API (`https://webhook.erply.com`).

## Auth
Send `clientCode` and `sessionKey` headers (obtained from classic `verifyUser`).

## Steps
1. `GET /v1/webhook-configuration` (`getWebhookConfigurations`) to see existing subscriptions.
2. `POST /v1/webhook-configuration` (`addWebhookConfiguration`) with a target URL, a table + action (`insert`/`update`/`delete`) or a named event, and a `secret`.
3. `PUT /v1/webhook-configuration/{id}` (`updateWebhookConfiguration`) / `DELETE /v1/webhook-configuration/{id}` (`removeWebhookConfiguration`) to change or remove it.
4. `GET /v1/event` (`getEvents`) to inspect recorded events; `GET /v1/webhook` (`getWebhooks`) to see recently sent deliveries.

## Rules
- Named events available: `posTransactionConfirmed`, `salesOrderCreated`, `salesOrderConfirmed` (see `asyncapi/erply-webhooks.yml`).
- Verify each delivery's HMAC using your configuration `secret` as the salt.
- One delivery batches up to 100 database events; your endpoint must return HTTP 200 within 5 seconds or the delivery is retried (default once) then dropped.
- Treat webhooks as a fire-and-forget notification signal, not an authoritative data source — reconcile with a PIM read.
