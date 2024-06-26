<p align="center">
  <img width="140" height="102" src="https://gfs-na.richardphotolab.com/img/logo/rpl-logo.png">
</p>

# Order/Shipped

- [Overview](#overview)
  - [Support](#support)
- [Method GET](#method-get)
  - [Response](#response)
- [Method POST](#method-post)
  - [Request](#request)
  - [Response](#response-1)

## Overview

The Shipped API is a private REST interface for approved Richard partners to manage newly shipped order notifications.

**Endpoint:** `/shipped`

The process is as follows:

- Partner polls the `shipped` endpoint using `GET` to be informed of orders Richard has shipped.
- Partner saves shipment data to their chosen storage solution.
- Partner acknowledges receipt of the notifications by calling the `shipped` endpoint via `POST`, removing it from the list.

The notification will only be removed from the list once it has been acknowledged as received. This puts most of the control for the flow of the process in the hands of the partner, allowing them to customize their implementation to their needs.

> :pushpin: Acknowledging notifications requires an absolute unique reference, so the `richardId` is used for this purpose. This id is available in the original `GET` request, to make it simple.

### Support

:point_right: Check out the [TESTING](TESTING.md) document for more information on testing your integration.

For API support, please email api.support@richardphotolab.com

## Method: `GET`

This method returns all orders currently marked as shipped _and_ not marked as received by you.

:warning: Keep in mind that every shipping notification must be acknowledged as received before it will be removed from the pending list. Therefore, if your pending and acknowledge processes are running out of sync, be sure to accommodate for possible duplicates in the pending list.

### Response

> :pushpin: This information is specific to this endpoint. You must *_also_* understand the [basic RESPONSE documentation](../RESPONSE.md).

#### Payload

| Field | Type | Description |
| ----- |:----:|-------------|
| `orders` | _array_ | List of orders |
| &nbsp;&nbsp;&nbsp;&nbsp;`index` | _integer_ | Internally generated index for each item |
| &nbsp;&nbsp;&nbsp;&nbsp;`mode` | _integer_ | Mode in which the request was made (`0`=Test / `1`=Live) |
| &nbsp;&nbsp;&nbsp;&nbsp;`richardId` | _string_ | Richard's internal identifier |
| &nbsp;&nbsp;&nbsp;&nbsp;`uniqueId` | _string_ \| _null_ | Provided unique id for this item |
| &nbsp;&nbsp;&nbsp;&nbsp;`orderNumber` | _string_ \| _null_ | Provided order number for this item |
| &nbsp;&nbsp;&nbsp;&nbsp;`shipment` | _object_ | Shipment |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`carrierProvider`    	   | _string_  | Carrier provider (e.g. UPS)    |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`carrierService`    	   | _string_  | Carrier service (e.g. "next day")    |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`trackingNumber`     | _string_  | Tracking number |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`datetime`           | _string_  | Date & time of shipment (Iso8601Zulu) |
| `errors` | _array_<_string_> | Error messages in the event that the order could not be processed

> :pushpin: All order responses include Richard's internal identifier. This is to provide a stable constant when handling possible issue resolution.

<br/>

HTTP Code: `200` (Success)

```JSON
{
  "orders": [
	{
      "index": 0,
	  "mode": 1,
      "richardId": "rpl-oae-ceb9fec4-a46c-4ead-99c2-1404b9ae82a6",
      "uniqueId": "A991321",
      "orderNumber": "RP9876",
	  "shipment": {
		"carrierProvider": "ups",
		"carrierService": "UPS Ground",
		"trackingNumber": "1Z001985YW99744790"
		"dateTime": "2022-10-02T13:28:12Z",
	  },
    },
    {
      "index": 1,
	  "mode": 1,
      "orderNumber": null,
      "uniqueId": null,
      "richardId": null,
      "errors": [    
	    "One of the order identification fields must be present",    
      ]
    },
  ],
  "errors": []
}
```
<br/>

## Method: `POST`

This method accepts an array of notification acknowledgements. The existence of an order in this list constitutes an acknowledgement that you previously received said notification.

A valid request payload consists of a base container _array_ containing response _objects_.

### Request

_object_

| Field             |   Type    | Required | Limits  | Description           |
| ----------------- | :-------: | :------: | :-----: | --------------------- |
| `richardId`       | _string_  |    Yes _(one of)_    | max 80  | Richard Identifier     |


```JSON
[
  {
	"richardId": "rpl-oae-ceb9fec4-a46c-8zad-09c2-1404b9ae812f"
  },
  {
	"richardId": "rpl-oae-ceb9fec4-a46c-4ead-99c2-1404b9ae82a6"
  }
]
```
<br/>

### Response

> :pushpin: This information is specific to this endpoint. You must *_also_* understand the [basic RESPONSE documentation](../RESPONSE.md).

_object_
Field | Type | Description
------|:----:|------------
`index` | _integer_ | Internally generated reference for each item
`mode` | _integer_ | Mode in which the request was made (`0`=Test / `1`=Live) |
`uniqueId` | _string_ \| _null_ | Provided unique id for this item
`accepted` | _integer_ (`0`/`1`) | Value representing if the order was accepted/successful
`richardId` | _string_ | Richard's internal identifier
`errors` | _array_<_string_> | Error messages in the event that the order count not be processed

> :pushpin: All order responses include Richard's internal identifier. This is to provide a stable constant when handling possible issue resolution.

#### 200 Created

HTTP Code: `200` (Success)

```JSON
{
  "orders": [
    {
      "index": 0,
	  "mode": 1,
      "richardId": "rpl-oae-ceb9fec4-a46c-4ead-99c2-1404b9ae82a6",
      "uniqueId": "A991321",
      "accepted": 1,
      "errors": []
    },
    {
      "index": 1,
	  "mode": 1,
      "richardId": "rpl-oae-hy9fec4-a46c-4ead-99c2-2404b9aeG4zS",
      "uniqueId": null,
      "accepted": 0,
      "errors": [    
	    "One of the order identification fields must be present",    
      ]
    },
    {
      "index": 2,
	  "mode": 1,
      "uniqueId": "A991328",
      "richardId": "rpl-oae-rsb9fec4-c76c-7fad-99c2-3724b9aeS9wM",
      "accepted": 1,
      "errors": []
    }
  ],
  "errors": []
}
```

:warning: ALL responses which are not system errors are returned with a `200` status code. This includes responses to requests where _no acknowledgements_ were accepted. Always check the `accepted` value for each order to know if it succeeded.
