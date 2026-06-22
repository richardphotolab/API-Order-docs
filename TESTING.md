<p align="center">
  <img width="140" height="102" src="https://gfs-na.richardphotolab.com/img/logo/rpl-logo.png">
</p>

# Testing

- [Overview](#overview)
  - [The Testbed Environment](#the-testbed-environment)
  - [Authentication](#authentication)
  - [The `mode` Field](#the-mode-field)
- [Order Lifecycle in the Testbed](#order-lifecycle-in-the-testbed)
  - [Creating Orders](#creating-orders)
  - [Shipping Your Orders](#shipping-your-orders)
  - [Retrieving Shipped Notifications](#retrieving-shipped-notifications)
  - [Tracking](#tracking)
- [Managing Your Testbed Data](#managing-your-testbed-data)

## Overview

Richard provides a dedicated **Testbed** environment so you can build and exercise your integration end-to-end without the risk of creating real orders. It is a complete mirror of the live API, with one crucial difference: **nothing you send is ever produced or shipped for real.** Orders, shipments, and tracking are all simulated.

### The Testbed Environment

A Richard contact will provide you with a testing API token and the testbed base URL.

> :pushpin: The testbed is reached at a different base URL than the live API. Both addresses are provided by your Richard technical contact. Point your integration at the testbed URL while developing, then switch to the live URL when you go to production.

The endpoints, request payloads, and response shapes are identical to live. Code written against the testbed will work unchanged against the live API.

### Authentication

Authentication is the same as the live API — supply your token in the `Authorization` header using the `Bearer` prefix. Your existing partner token works against the testbed; no new credentials are required.

:point_right: See [Request Basics](REQUEST.md#authentication) for details.

### The `mode` Field

The testbed processes **every** request as a test — there is nothing to opt into and no special token value or header to set.

Endpoints that support it return a `mode` field so you can confirm which environment handled your request. In the testbed this value is always `0` (test); the live API always returns `1`.

| `mode` | Meaning |
| :----: | ------- |
| `0`    | Test    |
| `1`    | Live    |

## Order Lifecycle in the Testbed

The testbed walks an order through the same lifecycle as production, so you can rehearse every step of your integration:

```
create  →  (ships)  →  appears in /shipped  →  you acknowledge  →  removed
```

### Creating Orders

Create orders by calling `POST /create` exactly as you would in production. The order is stored as a test order (`mode` `0`) with a status of `pending`.

:point_right: See the [Order/Create](endpoints/create.md) documentation for the request format.

### Shipping Your Orders

There are two ways an order gets shipped in the testbed — pick whichever suits the scenario you are testing:

- **Automatic (default).** Shortly after creation, the testbed simulates Richard fulfilling and shipping the order: it generates stand-in carrier and tracking data and marks the order as shipped. The order then appears in your `GET /shipped` results. You can adjust the delay or turn auto-shipping off entirely — see [Managing Your Testbed Data](#managing-your-testbed-data).
- **Manual.** Drive an order through its lifecycle yourself using the testbed dev endpoints. You can ship an order on demand, reset it back to `pending`, or delete it outright — handy for replaying a specific order number or testing a precise sequence of events.

> :pushpin: Whichever path you choose, the generated shipment data (carrier, service, tracking number) is simulated. It is shaped exactly like real shipment data so your parsing logic gets a faithful workout.

### Retrieving Shipped Notifications

Poll `GET /shipped` to receive the testbed orders that have shipped, just as you would in production. When you acknowledge an order via `POST /shipped`, it is flagged as received and removed from the pending list.

> :pushpin: Acknowledging an order still requires its `richardId`, exactly as in live. This `richardId` is present in the original `GET /shipped` response.

:point_right: See the [Order/Shipped](endpoints/shipped.md) documentation for more information.

### Tracking

`POST /tracking` returns **simulated** tracking information for testbed shipments. The response uses the same normalized, carrier-agnostic schema as the live endpoint, so you can build and verify your tracking integration against realistic data.

:point_right: See the [Order/Tracking](endpoints/tracking.md) documentation for the response schema.

## Managing Your Testbed Data

The testbed exposes a small set of **dev endpoints** that let you inspect and control your own test orders — list and view orders, ship/reset/delete them on demand, and configure auto-ship behavior. These are the tools you use to set up and tear down scenarios while developing.

:point_right: See the [Testbed Dev Endpoints](endpoints/dev.md) documentation for the full reference.

> :pushpin: The dev endpoints exist **only** in the testbed environment. They are not present on the live API.

---

For API support, please email api.support@richardphotolab.com
