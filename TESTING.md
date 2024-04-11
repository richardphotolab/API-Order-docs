<p align="center">
  <img width="140" height="102" src="https://gfs-na.richardphotolab.com/img/logo/rpl-logo.png">
</p>

# Testing

- [Overview](#overview)
  - [Eligibility](#eligibility)
- [Process](#process)
  - [Order Creation](#order-creation)
  - [Order Shipped](#order-shipped)


## Overview

A Richard contact will provide you with a testing API token. This token will allow you to test your integration with the Richard API. The testing environment is a mirror of the live environment, with the exception that no orders will be processed or shipped. This allows you to test your integration without the risk of creating real orders.

### Eligibility

A request is considered a test if it meets _any_ of the following criteria:

- The partner account is in test mode (managed by Richard).
- A test token is used for connecting (`rmd` value in the token).
- The `rpl-x-mode` header is set to `0` (zero).

## Process

The API will respond with a `mode` field in response payloads. When processing takes place in test mode, this value  will be `0`. This allows you to verify that your request was processed in the correct mode.

### Order Creation

When posting orders under a test mode, the API will automatically generate stand-in shipment data, and mark the order as complete. This facilitates testing of the `shipped` endpoint.

:point_right: Please see the [Order/Create](endpoints/create.md) documentation for more information.

### Order Shipped

When polling the `shipped` endpoint using test mode, you will receive a list of orders which were created in test mode. When you acknowledge these orders, they will be flagged as such and removed from the list. This allows you to test the process of receiving and acknowledging shipped orders in the same manner as live orders.

> :pushpin: Remember, when acknowledging orders you are still required to use the provided `richardId`. This applies to test orders as well.

:point_right: Please see the [Order/Shipped](endpoints/shipped.md) documentation for more information.