<p align="center">
  <img width="140" height="102" src="https://gfs-na.richardphotolab.com/img/logo/rpl-logo.png">
</p>

# Testbed Dev Endpoints

- [Overview](#overview)
  - [Availability](#availability)
  - [Authentication](#authentication)
  - [Response Format](#response-format)
- [Orders](#orders)
  - [List Orders — `GET /dev/orders`](#list-orders--get-devorders)
  - [Show Order — `GET /dev/order/{richardId}`](#show-order--get-devorderrichardid)
  - [Ship Order — `POST /dev/order/{richardId}/ship`](#ship-order--post-devorderrichardidship)
  - [Reset Order — `POST /dev/order/{richardId}/reset`](#reset-order--post-devorderrichardidreset)
  - [Delete Order — `DELETE /dev/order/{richardId}`](#delete-order--delete-devorderrichardid)
- [Partner](#partner)
  - [Show Partner — `GET /dev/partner/show`](#show-partner--get-devpartnershow)
  - [Update Options — `PATCH /dev/partner/options`](#update-options--patch-devpartneroptions)

## Overview

The dev endpoints are convenience tools for managing your own test data while building against the [Testbed](../TESTING.md). They let you inspect your orders, drive them through their lifecycle on demand (ship / reset / delete), and configure how the testbed auto-ships orders.

> :pushpin: New to the testbed? Start with the [TESTING](../TESTING.md) document for the bigger picture, then use this reference for the individual endpoints.

### Availability

These endpoints exist **only in the testbed environment**. They are not present on the live API. All actions are automatically scoped to your partner account — you can only see and affect your own orders. Requesting an order that does not belong to you returns a `404`.

### Authentication

Authentication is identical to the rest of the API: supply your token in the `Authorization` header with the `Bearer` prefix. Your existing partner token works here.

| Header          |   Type   | Required | Description                                               |
| --------------- | :------: | :------: | --------------------------------------------------------- |
| `authorization` | _string_ |   Yes    | Your token with the Bearer prefix (e.g. `Bearer <token>`) |

> :pushpin: Unlike `/create`, the dev endpoints do **not** require the `rpl-x-payloadchecksum` header. There is no signed payload to verify.

### Response Format

Every response is JSON and always includes an `errors` array (empty on success), following the [basic RESPONSE format](../RESPONSE.md). Successful responses also carry the relevant data under a named key (`orders`, `order`, `shipment`, `partner`, or `options`).

> :pushpin: The dev endpoints return raw record fields in `snake_case` (e.g. `richard_id`, `tracking_number`), which differ from the `camelCase` fields used by the main `create`/`shipped` endpoints. The `richard_id` value is the same identifier you know as `richardId` elsewhere.

#### Order object

| Field          |        Type        | Description                                                       |
| -------------- | :----------------: | ----------------------------------------------------------------- |
| `richard_id`   |      _string_      | Richard's internal identifier — use this in all other API calls   |
| `order_number` |      _string_      | The order number you provided on creation                         |
| `unique_id`    | _string_ \| _null_ | The unique id you provided on creation                            |
| `status`       |      _string_      | Lifecycle state: `pending`, `retrieved`, `complete`, or `shipped` |
| `mode`         | _integer_ (`0`/`1`) | Processing mode (`0` = Test). Always `0` in the testbed            |
| `created_at`   |      _string_      | When the order was created (ISO8601)                              |
| `id`           |     _integer_      | Internal reference. Prefer `richard_id` for identifying orders    |

#### Shipment object

| Field             |        Type        | Description                                  |
| ----------------- | :----------------: | -------------------------------------------- |
| `carrier`         |      _string_      | Simulated carrier (e.g. `usps`, `ups`)       |
| `carrier_service` | _string_ \| _null_ | Simulated carrier service (e.g. `UPS Ground`) |
| `tracking_number` |      _string_      | Simulated tracking number                    |
| `shipped_at`      |      _string_      | When the order shipped (ISO8601)             |
| `created_at`      |      _string_      | When the shipment record was created (ISO8601) |

## Orders

### List Orders — `GET /dev/orders`

Returns all of your testbed orders, most recent first.

**Response** — HTTP `200`

```JSON
{
  "orders": [
    {
      "id": 42,
      "created_at": "2026-06-21T13:20:01.000000Z",
      "mode": 0,
      "order_number": "PO12345678",
      "unique_id": "A991321",
      "status": "shipped",
      "richard_id": "rpl-oae-ceb9fec4-a46c-4ead-99c2-1404b9ae82a6"
    }
  ],
  "errors": []
}
```

### Show Order — `GET /dev/order/{richardId}`

Returns a single order along with its `shipment` (if any) and the stored order `payload`.

| Parameter    |   Type   | Description                            |
| ------------ | :------: | -------------------------------------- |
| `richardId`  | _string_ | The `richard_id` of the order to fetch |

**Response** — HTTP `200`

`shipment` is `null` until the order has shipped. `payload` is the order payload you originally submitted (see [Order/Create](create.md)); it is `null` if no payload is stored.

```JSON
{
  "order": {
    "id": 42,
    "created_at": "2026-06-21T13:20:01.000000Z",
    "mode": 0,
    "order_number": "PO12345678",
    "unique_id": "A991321",
    "status": "shipped",
    "richard_id": "rpl-oae-ceb9fec4-a46c-4ead-99c2-1404b9ae82a6"
  },
  "shipment": {
    "carrier": "usps",
    "carrier_service": "USPS Priority Mail",
    "tracking_number": "9400111899223344556677",
    "shipped_at": "2026-06-21T13:28:12.000000Z",
    "created_at": "2026-06-21T13:28:12.000000Z"
  },
  "payload": {
    "header": { "orderNumber": "PO12345678" }
  },
  "errors": []
}
```

**Not found** — HTTP `404`

```JSON
{
  "errors": [ "Order not found" ]
}
```

### Ship Order — `POST /dev/order/{richardId}/ship`

Immediately ships the order: generates simulated carrier and tracking data, creates the shipment record, and sets the order status to `shipped`. Use this to make an order appear in `GET /shipped` on demand, instead of waiting for (or in place of) auto-ship.

This action is **idempotent** — if the order is already shipped, the existing order and shipment are returned unchanged.

| Parameter    |   Type   | Description                           |
| ------------ | :------: | ------------------------------------- |
| `richardId`  | _string_ | The `richard_id` of the order to ship |

**Response** — HTTP `200`

```JSON
{
  "order": {
    "id": 42,
    "created_at": "2026-06-21T13:20:01.000000Z",
    "mode": 0,
    "order_number": "PO12345678",
    "unique_id": "A991321",
    "status": "shipped",
    "richard_id": "rpl-oae-ceb9fec4-a46c-4ead-99c2-1404b9ae82a6"
  },
  "shipment": {
    "carrier": "ups",
    "carrier_service": "UPS Ground",
    "tracking_number": "1Z001985YW99744790",
    "shipped_at": "2026-06-21T13:28:12.000000Z",
    "created_at": "2026-06-21T13:28:12.000000Z"
  },
  "errors": []
}
```

### Reset Order — `POST /dev/order/{richardId}/reset`

Resets the order back to `pending` and removes its shipment record. Use this to re-run an order through the shipping flow without recreating it.

| Parameter    |   Type   | Description                            |
| ------------ | :------: | -------------------------------------- |
| `richardId`  | _string_ | The `richard_id` of the order to reset |

**Response** — HTTP `200`

```JSON
{
  "order": {
    "id": 42,
    "created_at": "2026-06-21T13:20:01.000000Z",
    "mode": 0,
    "order_number": "PO12345678",
    "unique_id": "A991321",
    "status": "pending",
    "richard_id": "rpl-oae-ceb9fec4-a46c-4ead-99c2-1404b9ae82a6"
  },
  "errors": []
}
```

### Delete Order — `DELETE /dev/order/{richardId}`

Permanently deletes the order (and its shipment) from the testbed. Useful when you want to resubmit the same `orderNumber` from scratch.

| Parameter    |   Type   | Description                             |
| ------------ | :------: | --------------------------------------- |
| `richardId`  | _string_ | The `richard_id` of the order to delete |

**Response** — HTTP `200`

```JSON
{
  "errors": []
}
```

## Partner

### Show Partner — `GET /dev/partner/show`

Returns your partner profile together with your current testbed `options`.

**Response** — HTTP `200`

Each entry in `options` has a `key`, its `type`, and a normalized `value`. The testbed-specific options are described under [Update Options](#update-options--patch-devpartneroptions).

```JSON
{
  "partner": {
    "name": "Acme Photo Co.",
    "phone": "5551234567",
    "email": "integrations@acmephoto.example",
    "active": 1,
    "options": [
      {
        "key": "testbed_auto_ship",
        "type": "boolean",
        "value": true,
        "updated_at": "2026-06-21T13:00:00.000000Z"
      },
      {
        "key": "testbed_ship_delay",
        "type": "integer",
        "value": 30,
        "updated_at": "2026-06-21T13:00:00.000000Z"
      }
    ]
  },
  "errors": []
}
```

### Update Options — `PATCH /dev/partner/options`

Configures how the testbed auto-ships your orders. Send either or both fields; only the fields you send are changed.

#### Request

| Field                |        Type        | Required | Description                                                                                  |
| -------------------- | :----------------: | :------: | -------------------------------------------------------------------------------------------- |
| `testbed_auto_ship`  |     _boolean_      |    No    | Whether newly created orders are shipped automatically. Set `false` to ship manually instead |
| `testbed_ship_delay` | _integer_ \| _null_ |    No    | Delay in **seconds** before an order auto-ships (minimum `0`)                                 |

```JSON
{
  "testbed_auto_ship": true,
  "testbed_ship_delay": 30
}
```

#### Response — HTTP `200`

Echoes back the values that were applied.

```JSON
{
  "options": {
    "testbed_auto_ship": true,
    "testbed_ship_delay": 30
  },
  "errors": []
}
```

#### Validation Error — HTTP `422`

Returned when a value is the wrong type or out of range (e.g. a negative delay).

```JSON
{
  "errors": [ "The testbed ship delay field must be at least 0." ]
}
```

---

For API support, please email api.support@richardphotolab.com
