<p align="center">
  <img width="140" height="102" src="https://gfs-na.richardphotolab.com/img/logo/rpl-logo.png">
</p>

> :construction: **This endpoint is currently under development and is not yet available.** Do not attempt to integrate against it at this time. This documentation is provided as a preview only — details are subject to change. Contact api.support@richardphotolab.com if you have questions.

---

# Order/Tracking

- [Overview](#overview)
  - [How It Works](#how-it-works)
  - [Support](#support)
- [Authentication](#authentication)
- [Method: POST](#method-post)
  - [Request](#request)
  - [Carrier Resolution](#carrier-resolution)
  - [Response](#response)
- [Error Reference](#error-reference)

## Overview

The Tracking API is a private REST interface for approved Richard partners to look up real-time shipment tracking data for orders they have previously placed.

**Endpoint:** `/tracking`

### How It Works

1. After an order ships, the `/shipped` endpoint will notify you with the shipment details — including the tracking number and carrier.
2. You save that data on your end.
3. When you want to check the status of a shipment, `POST` to `/tracking` with the `richard_id` and `tracking_number`.
4. Richard queries the carrier on your behalf and returns the latest tracking data.

Responses are **cached for 5 minutes** on a per-partner basis. Repeated requests within that window return the cached result instantly (indicated by `"cached": true` in the response). This is transparent — the shape of the response is always the same.

> :pushpin: The `/tracking` endpoint accepts **`POST` only**. `GET` requests are not supported.

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

| Field            |    Type    | Required | Limits  | Description                                                                             |
| ---------------- | :--------: | :------: | :-----: | --------------------------------------------------------------------------------------- |
| `richard_id`     | _string_   |   Yes    | max 80  | Richard's internal order identifier (returned by `/create` and `/shipped`)              |
| `tracking_number`| _string_   |   Yes    |    ~    | The shipment tracking number (provided by `/shipped` when the order was shipped)        |
| `carrier`        | _string_   |    No    |    ~    | Carrier code (e.g. `usps`, `ups`, `stampscom`). Required for **live** orders — see [Carrier Resolution](#carrier-resolution) below. |

#### Request Example

```json
{
  "richard_id": "rpl-oae-ceb9fec4-a46c-4ead-99c2-1404b9ae82a6",
  "tracking_number": "9400111899220000000000",
  "carrier": "usps"
}
```

> :fire: **IMPORTANT — Why are both `richard_id` and `tracking_number` required?**
>
> - `richard_id` is used to verify that **you own the order** being queried. This prevents one partner from looking up another partner's shipment by guessing a tracking number.
> - `tracking_number` is passed directly to the carrier for the status lookup. For live orders, Richard does not store the tracking number server-side, so you must supply it. You received it from the `/shipped` endpoint when the order was shipped.
>
> Both fields are always required, regardless of order mode.

---

### Carrier Resolution

The `carrier` field controls which carrier API is queried. It is resolved in the following priority order:

| Priority | Source                                   | When available                                       |
| :------: | ---------------------------------------- | ---------------------------------------------------- |
| 1        | `carrier` field in your request body     | Always (if you provide it)                           |
| 2        | Test-mode carrier stored on the order    | **Test orders only** — omit `carrier` in test mode to use this |

> :warning: **Live orders require you to supply `carrier`.**
>
> For live orders, carrier data comes from an external shipping bridge and is not stored in Richard's system. You received the carrier code from the `/shipped` endpoint. If you omit `carrier` on a live order, the request will fail with `422`.

**Accepted carrier values** (case-insensitive):

| Value         | Carrier                  |
| ------------- | ------------------------ |
| `usps`        | USPS / Stamps.com (USPS) |
| `ups`         | UPS                      |
| `stampscom`   | Stamps.com               |
| `fedex`       | FedEx                    |

> :pushpin: The carrier value is normalized to lowercase before being forwarded to the carrier. You may submit it in any case (e.g. `UPS`, `ups`, or `Ups` are all equivalent).

---

### Response

> :pushpin: This information is specific to this endpoint. You must *_also_* understand the [basic RESPONSE documentation](../RESPONSE.md).

A successful response is always `HTTP 200`. The `cached` field indicates whether the result was served from cache (`true`) or freshly fetched from the carrier (`false`). The shape of `tracking` is identical in both cases.

#### Payload

_object_

| Field              |        Type        | Description                                                                               |
| ------------------ | :----------------: | ----------------------------------------------------------------------------------------- |
| `richard_id`       | _string_           | The `richard_id` you submitted in the request                                             |
| `tracking_number`  | _string_           | The `tracking_number` you submitted in the request                                        |
| `cached`           | _boolean_          | `true` if the result was served from cache; `false` if freshly fetched from the carrier   |
| `tracking`         | _object_           | Tracking data returned by the carrier — see structure below                               |
| `errors`           | _array_<_string_>  | Error messages (empty on success)                                                         |

#### `tracking` Object

| Field                                  |         Type         | Description                                                         |
| -------------------------------------- | :------------------: | ------------------------------------------------------------------- |
| `carrier`                              | _string_             | Carrier code (e.g. `usps`)                                          |
| `tracking_number`                      | _string_             | The tracking number that was queried                                 |
| `status`                               | _object_             | Current shipment status                                             |
| &nbsp;&nbsp;&nbsp;&nbsp;`category`     | _string_             | Status category (e.g. `in_transit`, `delivered`, `exception`)       |
| &nbsp;&nbsp;&nbsp;&nbsp;`summary`      | _string_             | Human-readable status summary (e.g. `"Delivered"`)                  |
| &nbsp;&nbsp;&nbsp;&nbsp;`detail`       | _string_             | Additional detail about the current status                          |
| `delivery`                             | _object_             | Delivery information                                                |
| &nbsp;&nbsp;&nbsp;&nbsp;`estimated_date` | _string_ \| _null_ | Estimated delivery date (`YYYY-MM-DD`), or `null` if unavailable    |
| &nbsp;&nbsp;&nbsp;&nbsp;`actual_date`  | _string_ \| _null_   | Actual delivery date (`YYYY-MM-DD`), or `null` if not yet delivered |
| &nbsp;&nbsp;&nbsp;&nbsp;`destination`  | _object_ \| _null_   | Destination address summary                                         |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`city`    | _string_ | Destination city                                            |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`state`   | _string_ | Destination state/province                                  |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`zip`     | _string_ | Destination postal code                                     |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`country` | _string_ | Destination country code                                    |
| `events`                               | _array_<_object_>    | Ordered list of tracking events (most recent first). May be empty.  |

> :pushpin: The `tracking` object is populated directly from the carrier's API. The exact content and availability of fields may vary by carrier and shipment state. Always guard against `null` values.

#### HTTP `200` — Success (cache miss)

```json
{
  "richard_id": "rpl-oae-ceb9fec4-a46c-4ead-99c2-1404b9ae82a6",
  "tracking_number": "9400111899220000000000",
  "cached": false,
  "tracking": {
    "carrier": "usps",
    "tracking_number": "9400111899220000000000",
    "status": {
      "category": "delivered",
      "summary": "Delivered",
      "detail": "Your item was delivered at 2:18 pm"
    },
    "delivery": {
      "estimated_date": null,
      "actual_date": "2026-04-29",
      "destination": {
        "city": "Portland",
        "state": "OR",
        "zip": "97201",
        "country": "US"
      }
    },
    "events": []
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
    "carrier": "usps",
    "tracking_number": "9400111899220000000000",
    "status": {
      "category": "in_transit",
      "summary": "In Transit",
      "detail": "Package is on its way"
    },
    "delivery": {
      "estimated_date": "2026-04-30",
      "actual_date": null,
      "destination": {
        "city": "Portland",
        "state": "OR",
        "zip": "97201",
        "country": "US"
      }
    },
    "events": []
  },
  "errors": []
}
```

---

## Error Reference

All non-system errors return `HTTP 200` with an `errors` array. System-level failures return a non-200 HTTP status code.

| HTTP Code | Meaning                         | Common Cause                                                                                                 |
| :-------: | ------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `400`     | Bad Request                     | `richard_id` or `tracking_number` is missing, or the payload is not valid JSON                              |
| `401`     | Unauthorized                    | Missing, invalid, or expired JWT token                                                                       |
| `404`     | Not Found                       | The order identified by `richard_id` does not exist, or it does not belong to your partner account           |
| `422`     | Unprocessable Entity            | `carrier` was not supplied and could not be determined (live orders always require `carrier`)                 |
| `502`     | Bad Gateway                     | Richard was unable to reach the carrier, or the carrier returned an error. Retry after a short delay.        |

> :warning: **`502` responses are transient.** They indicate that the tracking lookup could not be completed at this time — typically because the carrier API is temporarily unavailable. You should retry after a short delay. If the problem persists, contact api support.

> :pushpin: Error messages are intended for human consumption and **should not be parsed programmatically**. Always use HTTP status codes and the presence/content of the `errors` array to drive your application logic.


