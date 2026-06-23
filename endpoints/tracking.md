<p align="center">
  <img width="140" height="102" src="https://gfs-na.richardphotolab.com/img/logo/rpl-logo.png">
</p>

> :construction: **This endpoint is currently in beta.**. Contact api.support@richardphotolab.com if you have questions.

---

# Order/Tracking

- [Overview](#overview)
  - [How It Works](#how-it-works)
  - [Prerequisite: Retrieve Shipped Orders First](#prerequisite-retrieve-shipped-orders-first)
  - [Support](#support)
- [Authentication](#authentication)
- [Method: POST](#method-post)
  - [Request](#request)
  - [Response](#response)
    - [Status Values](#status-values)
    - [The `raw` Field](#the-raw-field)
- [Error Reference](#error-reference)

## Overview

The Tracking API is a private REST interface for approved Richard partners to look up real-time shipment tracking data for orders they have previously placed.

**Endpoint:** `/tracking`

### How It Works

1. After an order ships, you retrieve the shipment details from the [`/shipped`](shipped.md) endpoint. Doing so records the carrier and tracking number against the order inside Richard's system.
2. When you want to check the status of a shipment, `POST` to `/tracking` with **only** the `richard_id`.
3. Richard looks up the stored shipment for that order, resolves the carrier and tracking number itself, and queries the carrier on your behalf.
4. The carrier's response is normalized into a single, carrier-agnostic schema and returned to you.

Responses are **cached for 5 minutes** on a per-partner, per-order basis. Repeated requests within that window return the cached result instantly (indicated by `"cached": true` in the response). This is transparent — the shape of the response is always the same.

> :pushpin: The `/tracking` endpoint accepts **`POST` only**. `GET` requests are not supported.

### Prerequisite: Retrieve Shipped Orders First

Tracking only works for orders Richard has a **shipment record** for. That record is created when you retrieve the order through the [`/shipped`](shipped.md) endpoint — that is where Richard captures the carrier and tracking number.

If you call `/tracking` for an order that has no shipment record yet (e.g. it has not shipped, or you have never pulled it from `/shipped`), the request fails with `422` — see the [Error Reference](#error-reference). The fix is always the same: fetch the order from `/shipped` first, then retry.

In every case, the carrier and tracking number come from the stored shipment record, never from your request. In the [testbed](../TESTING.md), that record holds simulated data, so `/tracking` returns simulated tracking information.

### Support

:point_right: Check out the [TESTING](../TESTING.md) document for more information on testing your integration.

For API support, please email api.support@richardphotolab.com

---

## Authentication

> :pushpin: This endpoint uses the same token-based authentication as all other endpoints. See the [Request Basics](../REQUEST.md) documentation for full details.

You must include a valid `Authorization: Bearer <token>` header. The token is endpoint-specific — use the tracking token provided by your Richard technical contact.

The payload checksum header (`rpl-x-payloadchecksum`) is **not** required for this endpoint.

---

## Method: `POST`

### Request

The request body must be a **flat JSON object** (not an array). Only one order may be queried per request.

_object_

| Field         |    Type    | Required | Limits  | Description                                                                             |
| ------------- | :--------: | :------: | :-----: | --------------------------------------------------------------------------------------- |
| `richard_id`  | _string_   |   Yes    |    ~    | Richard's internal order identifier (returned by `/create` and `/shipped`). Identifies the order to look up. |
| `with_raw`    | _boolean_  |    No    |    ~    | When `true`, the unmodified carrier payload is included under `tracking.raw`. Defaults to `false`, in which case the `raw` key is **omitted entirely**. See [The `raw` Field](#the-raw-field). |

> :pushpin: The carrier and tracking number are **not** request fields — Richard resolves both from the order's stored shipment record. Any such keys you include in the body are ignored. See [How It Works](#how-it-works).

#### Request Example

```json
{
  "richard_id": "rpl-oae-ceb9fec4-a46c-4ead-99c2-1404b9ae82a6",
  "with_raw": false
}
```

> :fire: **Why is only `richard_id` needed?**
>
> - `richard_id` identifies the order **and** verifies that **you own it** — the lookup is scoped to your partner account.
> - The carrier and tracking number are resolved server-side from the shipment record Richard stored when you retrieved the order via [`/shipped`](shipped.md).

---

### Response

> :pushpin: This information is specific to this endpoint. You must *_also_* understand the [basic RESPONSE documentation](../RESPONSE.md).

A successful response is always `HTTP 200`. The `cached` field indicates whether the result was served from cache (`true`) or freshly fetched from the carrier (`false`). The shape of `tracking` is identical in both cases.

#### Payload

_object_

| Field              |        Type        | Description                                                                               |
| ------------------ | :----------------: | ----------------------------------------------------------------------------------------- |
| `richard_id`       | _string_           | The `richard_id` you submitted in the request                                             |
| `tracking_number`  | _string_           | The tracking number Richard looked up — resolved from the order's shipment record         |
| `cached`           | _boolean_          | `true` if the result was served from cache; `false` if freshly fetched from the carrier   |
| `tracking`         | _object_           | Normalized tracking data — see structure below                                             |
| `errors`           | _array_<_string_>  | Error messages (empty on success)                                                         |

#### `tracking` Object

The `tracking` object is a **carrier-agnostic, normalized** view of the shipment, produced by Richard. The same field set is returned regardless of which carrier actually fulfilled the shipment — USPS, UPS, and Stamps.com responses are all mapped into this single schema, so you only have to integrate once. Richard returns this normalized object verbatim; carrier-specific quirks are absorbed by the normalization layer before they reach you.

| Field                  |         Type         | Description                                                                                          |
| ---------------------- | :------------------: | ---------------------------------------------------------------------------------------------------- |
| `tracking_number`      | _string_             | The tracking number that was queried.                                                                |
| `carrier`              | _string_             | Carrier code that fulfilled the shipment: `usps`, `ups`, or `stampscom`.                              |
| `service_type`         | _string_ \| _null_   | Carrier service level, verbatim from the carrier (e.g. `PRIORITY_MAIL`, `UPS Ground`). May be `null`.|
| `status`               | _string_             | Normalized status. One of a fixed set — see [Status Values](#status-values).                          |
| `is_delivered`         | _boolean_            | Convenience flag. `true` only when `status` is `delivered`.                                           |
| `is_exception`         | _boolean_            | Convenience flag. `true` only when `status` is `exception`. (A `returned` shipment is **not** an exception.) |
| `ship_date`            | _string_ \| _null_   | Date the package shipped (`YYYY-MM-DD`), or `null` if the carrier does not report it.                 |
| `estimated_delivery`   | _object_ \| _null_   | Estimated delivery window, or `null` if unavailable. See below.                                       |
| &nbsp;&nbsp;&nbsp;&nbsp;`begins` | _string_ \| _null_ | Earliest estimated delivery date (`YYYY-MM-DD`).                                            |
| &nbsp;&nbsp;&nbsp;&nbsp;`ends`   | _string_ \| _null_ | Latest estimated delivery date (`YYYY-MM-DD`). When the carrier gives a single date, `begins` and `ends` are equal. |
| `actual_delivery_date` | _string_ \| _null_   | Actual delivery date (`YYYY-MM-DD`), or `null` if not yet delivered.                                  |
| `origin`               | _object_ \| _null_   | Origin address (see [Address Object](#address-object)). Frequently `null` — carriers rarely expose it on tracking. |
| `destination`          | _object_ \| _null_   | Destination address (see [Address Object](#address-object)).                                          |
| `signed_by`            | _string_ \| _null_   | Name of the person who signed for the package, when applicable. Usually `null`.                       |
| `events`               | _array_<_object_>    | Tracking events in the order the carrier returns them (typically most recent first). May be empty.   |
| &nbsp;&nbsp;&nbsp;&nbsp;`occurred_at`  | _string_   | Timestamp of the event (`YYYY-MM-DDTHH:MM:SS`, carrier-local time).                          |
| &nbsp;&nbsp;&nbsp;&nbsp;`description`  | _string_   | Human-readable event description, verbatim from the carrier.                                 |
| &nbsp;&nbsp;&nbsp;&nbsp;`status`       | _string_   | Normalized status for this event — same vocabulary as the top-level `status`.                |
| &nbsp;&nbsp;&nbsp;&nbsp;`location`     | _object_ \| _null_ | Where the event occurred (see [Address Object](#address-object); city/province/postal/country only). |
| `raw`                  | _object_             | **Only present when `with_raw: true` was sent.** The unmodified carrier payload. See [The `raw` Field](#the-raw-field). |

##### Address Object

`origin`, `destination`, and event `location` share the same structure (event `location` omits the street lines):

| Field          |       Type        | Description                |
| -------------- | :---------------: | -------------------------- |
| `street`       | _string_ \| _null_ | Street line 1             |
| `street2`      | _string_ \| _null_ | Street line 2             |
| `city`         | _string_ \| _null_ | City                      |
| `province`     | _string_ \| _null_ | State / province          |
| `postal_code`  | _string_ \| _null_ | Postal / ZIP code         |
| `country_code` | _string_ \| _null_ | ISO country code          |

> :pushpin: **Every field except `tracking_number`, `carrier`, `status`, `is_delivered`, and `is_exception` may be `null` (or an empty `events` array).** What a carrier reports varies by carrier and by where the package is in its journey. Normalization guarantees a consistent *shape*, not that every field is populated. Always guard against `null`.

#### Status Values

The `status` field (both top-level and per-event) is always one of these values:

| Value                | Meaning                                                                       |
| -------------------- | ----------------------------------------------------------------------------- |
| `accepted`           | The carrier has accepted / picked up the package.                              |
| `in_transit`         | The package is moving through the carrier network.                             |
| `out_for_delivery`   | The package is out for delivery today.                                         |
| `delivery_attempted` | A delivery attempt was made but the package was not delivered.                 |
| `delivered`          | The package was delivered.                                                     |
| `exception`          | A problem occurred (e.g. delay, damage, or an address issue).                  |
| `returned`           | The package is being returned, or has been returned, to the sender.           |
| `unknown`            | The status could not be determined from the carrier's data.                   |

> :pushpin: Treat this as a **closed set**, but code defensively: if we add a new status in the future, fall back to handling it like `unknown` rather than erroring.

#### HTTP `200` — Success (cache miss)

```json
{
  "richard_id": "rpl-oae-ceb9fec4-a46c-4ead-99c2-1404b9ae82a6",
  "tracking_number": "9400111899220000000000",
  "cached": false,
  "tracking": {
    "tracking_number": "9400111899220000000000",
    "carrier": "usps",
    "service_type": "PRIORITY_MAIL",
    "status": "delivered",
    "is_delivered": true,
    "is_exception": false,
    "ship_date": "2026-04-27",
    "estimated_delivery": {
      "begins": "2026-04-29",
      "ends": "2026-04-29"
    },
    "actual_delivery_date": "2026-04-29",
    "origin": null,
    "destination": {
      "street": null,
      "street2": null,
      "city": "Portland",
      "province": "OR",
      "postal_code": "97201",
      "country_code": "US"
    },
    "signed_by": "J SMITH",
    "events": [
      {
        "occurred_at": "2026-04-29T14:18:00",
        "description": "Delivered, In/At Mailbox",
        "status": "delivered",
        "location": {
          "city": "Portland",
          "province": "OR",
          "postal_code": "97201",
          "country_code": "US"
        }
      },
      {
        "occurred_at": "2026-04-29T08:00:00",
        "description": "Out for Delivery",
        "status": "out_for_delivery",
        "location": null
      }
    ]
  },
  "errors": []
}
```

#### HTTP `200` — Success (cache hit)

The response shape is identical — only `"cached": true` differs.

```json
{
  "richard_id": "rpl-oae-ceb9fec4-a46c-4ead-99c2-1404b9ae82a6",
  "tracking_number": "9400111899220000000000",
  "cached": true,
  "tracking": {
    "tracking_number": "9400111899220000000000",
    "carrier": "usps",
    "service_type": "PRIORITY_MAIL",
    "status": "in_transit",
    "is_delivered": false,
    "is_exception": false,
    "ship_date": "2026-04-27",
    "estimated_delivery": {
      "begins": "2026-04-30",
      "ends": "2026-04-30"
    },
    "actual_delivery_date": null,
    "origin": null,
    "destination": {
      "street": null,
      "street2": null,
      "city": "Portland",
      "province": "OR",
      "postal_code": "97201",
      "country_code": "US"
    },
    "signed_by": null,
    "events": []
  },
  "errors": []
}
```

#### The `raw` Field

By default the `raw` field is **omitted entirely** from the `tracking` object. To receive it, send `"with_raw": true` in your request. When present, `raw` contains the carrier's **unmodified** response payload exactly as Richard received it from the carrier.

> :warning: **`raw` is outside of Richard's control; use with caution.**
>
> - Its shape is **carrier-specific** and completely different between USPS, UPS, FedEx and Stamps.com.
> - It is **not versioned** and may change at any time without notice when a carrier changes their API.
> - All the data you need for normal operation should already in the normalized fields above.

> :pushpin: Raw and non-raw responses are cached separately, so toggling `with_raw` always reflects your current request rather than a previously cached variant.

> :test_tube: **Testing with `with_raw` on the testbed:** When you send `with_raw: true` in the testbed, the `raw` field is present and is an object, but its contents are a lightweight stub — it contains only `carrier`, `tracking_number`, and a fixed `status` string drawn from the stored shipment record. This is enough to verify that your integration correctly handles the presence of `raw`, but it does not replicate the actual shape a real carrier returns. Carrier-specific `raw` payloads differ significantly between USPS, UPS, and Stamps.com, and none of that structure can be faithfully simulated. To understand what `raw` looks like for a specific carrier in production, consult that carrier's public tracking API documentation.

```json
{
  "richard_id": "rpl-oae-ceb9fec4-a46c-4ead-99c2-1404b9ae82a6",
  "tracking_number": "9400111899220000000000",
  "cached": false,
  "tracking": {
    "tracking_number": "9400111899220000000000",
    "carrier": "usps",
    "service_type": "PRIORITY_MAIL",
    "status": "in_transit",
    "is_delivered": false,
    "is_exception": false,
    "ship_date": "2026-04-27",
    "estimated_delivery": { "begins": "2026-04-30", "ends": "2026-04-30" },
    "actual_delivery_date": null,
    "origin": null,
    "destination": { "street": null, "street2": null, "city": "Portland", "province": "OR", "postal_code": "97201", "country_code": "US" },
    "signed_by": null,
    "events": [],
    "raw": {
      "statusCategory": "In Transit",
      "mailClass": "PRIORITY_MAIL",
      "trackingEvents": [ "...carrier-specific fields..." ]
    }
  },
  "errors": []
}
```

---

## Error Reference

All non-system errors return `HTTP 200` with an `errors` array. System-level failures return a non-200 HTTP status code.

| HTTP Code | Meaning                         | Common Cause                                                                                                 |
| :-------: | ------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `400`     | Bad Request                     | `richard_id` is missing, `with_raw` is not a boolean, the payload is empty, or the payload is not valid JSON  |
| `401`     | Unauthorized                    | Missing, invalid, or expired JWT token; or your partner account is inactive                                   |
| `404`     | Not Found                       | The order identified by `richard_id` does not exist, or it does not belong to your partner account           |
| `422`     | Unprocessable Entity            | The order has no shipment record yet. Retrieve it via [`/shipped`](shipped.md) first, then retry.            |
| `502`     | Bad Gateway                     | The tracking lookup could not be completed — Richard was unable to reach the carrier service, or the carrier returned an error. Retry after a short delay. |

> :warning: **`502` responses are transient.** They indicate that the tracking lookup could not be completed at this time — typically because the carrier (or the carrier service) is temporarily unavailable, or the carrier could not resolve the tracking number. You should retry after a short delay. If the problem persists, contact api support.

> :pushpin: Error messages are intended for human consumption and **should not be parsed programmatically**. Always use HTTP status codes and the presence/content of the `errors` array to drive your application logic.


