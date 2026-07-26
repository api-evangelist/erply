---
name: Sync products from Erply PIM
description: Authenticate to Erply and page through the product catalog (products, categories, groups) to sync into an external system such as an ecommerce store.
api: openapi/erply-pim-openapi.json
operations:
  - "POST verifyUser (classic auth)"
  - "GET /v1/product"
  - "GET /v1/product/{ids}"
  - "GET /v1/product/category"
  - "GET /v1/product/group"
---

# Sync products from Erply PIM

Use this skill to pull the Erply product catalog into an external system.

## Auth
1. Call the classic API `verifyUser` (POST username + password + clientCode) to obtain a `sessionKey`. Sessions last ~1 hour; error `1054`/`1055` means re-authenticate.
2. Send `clientCode` and `sessionKey` headers on every PIM REST call (`https://api-pim-<region>.erply.com`).

## Steps
1. `GET /v1/product/category` and `GET /v1/product/group` to build the taxonomy.
2. Page the catalog with `GET /v1/product` using `take` / `skip`; pass `withTotalCount=1` on the first page to learn the total.
3. For detail on a known set, `GET /v1/product/{ids}` (batch).
4. Map fields and upsert into the target system.

## Rules
- Respect the rate limit: 2000 requests/hour/account — error `1002` on breach. Back off until the next hour.
- Bulk reads (`/v1/product/bulk/get`) cap at 100 sub-requests (error `1020`).
- Errors return the `MessageResponse` schema on 4xx/5xx; the classic API returns a `status.errorCode` (see `errors/erply-error-codes.yml`).
- Do not poll aggressively; prefer webhooks (see `skills/erply-manage-webhooks.md`) for change notification.
