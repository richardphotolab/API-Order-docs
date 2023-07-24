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

The Shipped API is a private REST interface for approved Richard partners to query for pending newly shipped orders.

### Support

For API support, please email api.support@richardphotolab.com

## Method: `GET`

This method returns all orders currently marked as shipped _and_ not marked as received by you.
> :warning: Keep in mind that every shipping notification must be acknowledged as received before it will be removed from the pending list. Therefore, if your pending and acknowledge processes are running out of sync, be sure to accommodate for possible duplicates in the pending list.

### Response

> :warning: This information is specific to this endpoint. You must *_also_* understand the [basic RESPONSE documentation](../REQUEST.md).

#### Payload

| Field | Type | Description |
| ----- |:----:|-------------|
| `index` | _integer_ | Internally generated reference for each item
| `orderNumber` | _string_ \| _null_ | Provided order number for this item
| `uniqueId` | _string_ \| _null_ | Provided unique id for this item
| `richardId` | _string_ | Richard's internal identifier
| `shipment` | _array_ | Shipment
| &nbsp;&nbsp;&nbsp;&nbsp;_(recurring object)_                     | _object_  |   |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`carrierName`    | _string_  | Carrier name    |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`trackingNumber` | _string_  | Tracking number |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`datetime`       | _string_  | Date & time of shipment (Iso8601Zulu) |
`errors` | _array_<_string_> | Error messages in the event that the order could not be processed

> :warning: All order responses include Richard's internal identifier. This is to provide a stable constant when handling possible issue resolution.

<br/>

HTTP Code: `200` (Success)

```JSON
{
  "orders": [
    {
      "index": 0,
      "orderNumber": "RP9876",
      "uniqueId": "A991321",
      "richardId": "rpl-oae-ceb9fec4-a46c-4ead-99c2-1404b9ae82a6",
	  "shipment": {
		"datetime": "2022-10-02T13:28:12Z",
		"carrier": "ups",
		"trackingNumber": "1Z001985YW99744790"
	  },
      "errors": []
    },
    {
      "index": 1,
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
| `orderNumber`     | _string_  |    Yes _(one of)_    | max 20  | Order Number          |
| `uniqueId`        | _string_  |    Yes _(one of)_    | max 80  | Unique Identifier     |
| `richardId`       | _string_  |    Yes _(one of)_    | max 80  | Richard Identifier     |

> :raised_hand: Only one of the order identification fields above is required to acknowledge an order.

```JSON
[
  {
    "orderNumber": "RP9876",
  },
  {
	"uniqueId": "154687354234"
  },
  {
	"richardId": "rpl-oae-ceb9fec4-a46c-4ead-99c2-1404b9ae82a6"
  }
]
```
<br/>

### Response

> :warning: This information is specific to this endpoint. You must *_also_* understand the [basic RESPONSE documentation](../REQUEST.md).

_object_
Field | Type | Description
------|:----:|------------
`index` | _integer_ | Internally generated reference for each item
`orderNumber` | _string_ \| _null_ | Provided order number for this item
`uniqueId` | _string_ \| _null_ | Provided unique id for this item
`accepted` | _integer_ (`0`/`1`) | Value representing if the order was accepted/successful
`richardId` | _string_ | Richard's internal identifier
`errors` | _array_<_string_> | Error messages in the event that the order count not be processed

> :ok_hand: All order responses include Richard's internal identifier. This is to provide a stable constant when handling possible issue resolution.

#### 200 Created

HTTP Code: `200` (Success)

```JSON
{
  "orders": [
    {
      "index": 0,
      "orderNumber": "RP9876",
      "uniqueId": "A991321",
      "richardId": "rpl-oae-ceb9fec4-a46c-4ead-99c2-1404b9ae82a6",
      "accepted": 1,
      "errors": []
    },
    {
      "index": 1,
      "orderNumber": "RP9879",
      "uniqueId": null,
      "richardId": "rpl-oae-hy9fec4-a46c-4ead-99c2-2404b9aeG4zS",
      "accepted": 0,
      "errors": [    
	    "One of the order identification fields must be present",    
      ]
    },
    {
      "index": 2,
      "orderNumber": "RP9880",
      "uniqueId": "A991328",
      "richardId": "rpl-oae-rsb9fec4-c76c-7fad-99c2-3724b9aeS9wM",
      "accepted": 1,
      "errors": []
    }
  ],
  "errors": []
}
```

> :warning: ALL responses which are not system errors are returned with a `200` status code. This includes responses to requests where _no acknowledgements_ were accepted. Always check the `accepted` value for each order to know if it succeeded.
