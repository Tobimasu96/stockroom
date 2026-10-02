# API examples — Chapter 2

Status: proposed contract for review. These endpoints are not implemented. GBP is the agreed single currency. This document collects the examples discussed with the learner; it is not yet the complete OpenAPI specification.

## Common behavior

- Base path: `/api/v1`.
- JSON request/response DTOs, never persistence entities or password hashes.
- Session authentication; cookie-authenticated writes require a valid CSRF token. Examples omit login/cookie/CSRF details until Chapter 4.
- ADMIN manages product writes. STAFF handles stock and orders. The exact ADMIN inheritance and read-permission matrix remain to be defined.
- Monetary values use Java BigDecimal and GBP. Proposed validation: nonnegative prices, at most two decimal places, no silent rounding. Maximum monetary value remains undecided.
- Illustrative IDs below are numeric; the actual ID type remains undecided.
- Proposed status codes: 400 invalid input, 401 missing authentication, 403 insufficient role or failed CSRF, 404 missing resource, 409 conflict. Exact security responses and the stable error schema need to be specified and tested.

## Create product

`POST /api/v1/products` — ADMIN

```json
{
  "sku": "BOOK-RED",
  "name": "Red Book",
  "price": 2.50,
  "lowStockThreshold": 5
}
```

Proposed rules: SKU/name required and nonblank; SKU unique including archived products; price required, nonnegative, with at most two decimal places; threshold required and a nonnegative integer. SKU normalization and field lengths remain undecided.

Success: `201 Created`, `Location: /api/v1/products/101`, and:

```json
{
  "id": 101,
  "sku": "BOOK-RED",
  "name": "Red Book",
  "price": 2.50,
  "active": true,
  "lowStockThreshold": 5
}
```

Proposed initialization: create the product and its zero-quantity inventory record atomically. Initial goods are received via audited stock adjustments. Duplicate SKU returns 409; invalid fields return 400. JSON numeric formatting does not guarantee trailing zeros; 2.5 and 2.50 represent the same monetary value.

## Receive or adjust stock

`POST /api/v1/products/101/stock-adjustments` — STAFF

```json
{
  "quantityChange": 10,
  "reason": "STOCK_RECEIVED"
}
```

Success: proposed `201 Created` with the created movement:

```json
{
  "id": 501,
  "productId": 101,
  "quantityChange": 10,
  "reason": "STOCK_RECEIVED",
  "actorId": 7,
  "occurredAt": "2026-10-02T10:00:00Z"
}
```

The server derives actor and UTC timestamp. Inventory update and movement creation share a transaction. Proposed validation: nonzero integer change; STOCK_RECEIVED requires a positive change. Other adjustment reasons and archived-product behavior need definition. Missing product returns 404; a result below zero returns 409. Stock-adjustment retry deduplication is not yet specified: an order's idempotency guarantee must not be assumed for this endpoint.

## Place order

`POST /api/v1/orders` — STAFF

Header: `Idempotency-Key: red-book-order-001`

```json
{
  "lines": [
    { "productId": 101, "quantity": 4 }
  ]
}
```

The client supplies quantities, not prices. The server looks up and snapshots current prices. Success: proposed `201 Created`, `Location: /api/v1/orders/1001`, and:

```json
{
  "id": 1001,
  "status": "PLACED",
  "currency": "GBP",
  "createdAt": "2026-10-02T10:05:00Z",
  "lines": [
    {
      "productId": 101,
      "quantity": 4,
      "unitPrice": 2.50,
      "lineTotal": 10.00
    }
  ],
  "total": 10.00
}
```

Ten available books become six. Order, lines, price snapshots, stock, movements, idempotency result, and applicable outbox events commit together or all roll back.

| Situation | Proposed response |
| --- | --- |
| Empty lines, nonpositive/noninteger quantity, missing key | 400 |
| Referenced product missing | 404 |
| Insufficient stock or inactive product | 409 |
| Same key and same payload, including concurrent retries | Original order; proposed replay status 201 |
| Same key with different payload | 409 |

Clients generate and retain the key/payload before sending, then reuse them after a connection failure. A new key is treated as a new request. Key scope/length/retention and canonical payload comparison remain to be specified. Duplicate-line combination is proposed in the domain model but not finalized.

## Cancel or fulfil order

- `POST /api/v1/orders/1001/cancellation` — STAFF, no request body.
- `POST /api/v1/orders/1001/fulfilment` — STAFF, no request body.

Success: proposed `200 OK` with the order response above and updated status. Only PLACED may transition to CANCELLED or FULFILLED. Cancelling restores every line's stock once and records movements; fulfilling does not deduct stock again. A repeated cancellation returns the already-cancelled order with 200 and no additional restoration. Proposed repeated fulfilment behaves similarly. Missing order returns 404; a forbidden transition returns 409.

Serialize concurrent fulfilment/cancellation so only one valid terminal state wins. Status, stock changes, and audit history are transactional.

## Remaining contract work

Define product reads/updates/archive, order reads/filtering, low-stock reports, bounded pagination, the full permission matrix, stable error bodies, request size bounds, and unresolved validation/idempotency choices. Then write and validate the complete OpenAPI specification. Chapter 2 is not complete on the basis of these examples.

Official learning resource: [MDN HTTP overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview). Exercise: compare the create-product and create-order payloads; explain why the order request should not accept a client-selected unit price.
