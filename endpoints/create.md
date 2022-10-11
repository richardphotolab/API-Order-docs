<p align="center">
  <img width="140" height="102" src="https://www.richardphotolab.com/themes/rpl/assets/img/rpl-logo.png">
</p>

# Order/Create

- [Overview](#overview)
  - [Support](#support)
  - [Anatomy](#anatomy)
- [Authorization](#authorization)
  - [Headers](#headers)
- [Payload](#payload)
- [Responses](#responses)
  - [Service Errors](#service-errors)
  - [Orders](#orders)
  - [Errors](#errors)



## Overview

The Create API is a private REST interface for approved Richard partners to submit new order registrations for processing.

### Support

For API support, please email api.support@richardphotolab.com

## Payload

> :warning: This information is specific to this endpoint. You must *_also_* understand the [base request documentation](../REQUEST.md).

The payload is designed to provide the ability to include one or many order registrations, as needed. A payload consists of a base container _array_ containing order _objects_. There must be a minimum of one(1) order, and a maximum of fifty(50).

### Requests

**Method:** `POST`

_object_

| Field                                                            |   Type    | Required | Limits  | Description           |
| ---------------------------------------------------------------- | :-------: | :------: | :-----: | --------------------- |
| `header`                                                         | _object_  |   Yes    |    ~    | Order header          |
| &nbsp;&nbsp;&nbsp;&nbsp;`orderNumber`                            | _string_  |   Yes    | max 20  | Order Number          |
| &nbsp;&nbsp;&nbsp;&nbsp;`uniqueId`                               | _string_  |    No    | max 80  | Unique Identifier     |
| &nbsp;&nbsp;&nbsp;&nbsp;`orderReference`                         | _string_  |    No    | max 50  | Reference ID          |
| &nbsp;&nbsp;&nbsp;&nbsp;`orderPromoCode`                         | _string_  |    No    | max 50  | Promotion Code        |
| &nbsp;&nbsp;`shipping`                                           | _object_  |   Yes    |    ~    |                       |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`customerName`   | _string_  |   Yes    | max 100 | Shipping Name         |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`address1`       | _string_  |   Yes    | max 100 | Shipping Address 1    |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`address2`       | _string_  |   Yes    | max 100 | Shipping Address 2    |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`city`           | _string_  |   Yes    | max 50  | Shipping City         |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`province`          | _string_  |   Yes    | max 50  | Shipping Province/State        |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`postalCode`     | _string_  |   Yes    | max 16  | Shipping Postal Code  |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`country`        | _string_  |   Yes    | max 50  | Shipping Country      |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`phone`          | _integer_ |   Yes    | max 18  | Shipping Phone Number |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`shippingMethod` | _string_  |   Yes    | max 20  | Shipping Method       |
| `items`                                                          |  _array_  |   Yes    |    ~    | Items                 |
| &nbsp;&nbsp;&nbsp;&nbsp;_(recurring object)_                     | _object_  |   Yes    |    ~    |                       |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`code`       | _string_  |   Yes    | max 50  | Item Code             |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`quantity`       | _integer_ |   Yes    |  max 3  | Item Quantity         |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`sourceImage`    | _string_  |    No    |   URL   | Source Image URL      |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`options`        | _array_<_object_>  |    No    |    ~    | Item Options          |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;_(recurring object)_ | _object_  |   No    |   ~  |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`code` | _string_ | Yes |  ~ | Option Code   |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`quantity`  | _integer_ |   No    |  ~  | Option Quantity         |
| `options`                                                        | _array_<_object_> |    No    |    ~    | Options               |
| &nbsp;&nbsp;&nbsp;&nbsp;_(recurring object)_                     | _object_  |    No    |    ~    |						|
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`code`           | _string_  |   Yes    |    ~    | Option Code             |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`quantity`       | _integer_ |    No    |    ~    | Option Quantity         |

```JSON
[
  {
    "header": {
      "orderNumber": "PO0061",
      "shipping": {
        "customerName": "Partner Customer",
        "address1": "123 Street Address",
        "address2": "Apt #6",
        "city": "Salt Lake City",
        "province": "Utah",
        "postalCode": "11111",
        "country": "USA",
        "phone": 1111111111,
        "shippingMethod": "SPSSP"
      }
    },
    "items": [
      {
        "code": "CP1010P",
        "quantity": 1,
        "sourceImage": "https://orders-sf1-api.richardphotolab.com/storage/sample-image.jpg",
        "options": [
          {
            "code": "MT1020S2",
            "quantity": 1
          }
        ]
      },
      {
        "code": "CP1020D",
        "quantity": 1,
        "sourceImage": "https://orders-sf1-api.richardphotolab.com/storage/sample-image.jpg",
        "cropDetails": "[0.2, 0.1, 0.95, 0.85]",
        "orientation": 90
      },
      {
        "code": "CP1010P",
        "quantity": 1,
        "sourceImage": "https://orders-sf1-api.richardphotolab.com/storage/sample-image.jpg",
        "options": [
          {
            "code": "MT1020S2",
            "quantity": 2
          }
        ]
      }
    ],
    "options": [
      {
        "code": "SPCC",
        "quantity": 2
      },
      {
        "code": "SPEP",
        "quantity": 1
      }
    ]
  }
]
```

<br/>

## Responses

All responses from the service will be formatted as valid JSON.

The response will include an `errors` array containing any errors which occurred during the processing of your request, and an `orders` array containing details of each item in your request (unless a Service Error).

All response payloads except Service Errors (see below), also include a `mode` Boolean integer field indicating if the request payload was processed in a test or live mode.

### Service Errors

Service errors are low level failures which block any further processing of the request payload. These include authorization failures as well as malformed payloads (invalid JSON structure, missing required fields). The type of error is determined by examining the return HTTP status code along with the error message(s) in the returned payload.

The payload is made up of an `errors` array containing a message line for each error which occurred.

> :ok_hand: In most circumstances service errors generate only one message line

_object_
Field | Type | Description
------|:----:|------------
`errors` | _array_<_string_> | Messages of the error which occurred

#### 401 Unauthorized

HTTP Code: `401`

```JSON
{
  "errors": [ "Authorization failure" ]
}
```

#### 422 Unprocessable Entity

HTTP Code: `422`

```JSON
{
  "errors": [ "Malformed payload" ]
}
```

#### 500 Internal Error

HTTP Code: `500` :scream:

```JSON
{
  "errors": [ "Unknown error" ]
}
```

> :boom: Unknown errors should _never_ occur. If you receive this response please cease sending requests to the service and contact api support immediately.

<br/>

### Orders

_object_
Field | Type | Description
------|:----:|------------
`index` | _integer_ | Internally generated reference for each item
`orderNumber` | _string_ \| _null_ | Provided order number for this item
`uniqueId` | _string_ \| _null_ | Provided unique id for this item
`accepted` | _integer_ (`0`/`1`) | Value representing if the order was accepted/successful
`richardId` | _string_ | Richard's internal identifier
`createdAt` | _string_ \| _null_ | Full string date and time in UTC when the order was accepted
`errors` | _array_<_string_> | Error messages in the event that the order count not be processed

> :warning: All order responses include Richard's internal identifier. This is to provide a stable constant when handling possible issue resolution.

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
      "createdAt": "2019-11-27T16:26:59.000000Z",
      "errors": []
    },
    {
      "index": 1,
      "orderNumber": "RP9879",
      "uniqueId": null,
      "richardId": "rpl-oae-hy9fec4-a46c-4ead-99c2-2404b9aeG4zS",
      "accepted": 0,
      "createdAt": null,
      "errors": [    
	    "The header.customer.email must be a valid email address",    
	    "The items.1.quantity field is required"
      ]
    },
    {
      "index": 2,
      "orderNumber": "RP9880",
      "uniqueId": "A991328",
      "richardId": "rpl-oae-rsb9fec4-c76c-7fad-99c2-3724b9aeS9wM",
      "accepted": 1,
      "createdAt": "2019-11-27T16:27:32.000000Z",
      "errors": []
    }
  ],
  "errors": []
}
```

> :warning: All responses which are not system errors are returned with a `200` status code. This includes responses to requests where _no orders_ were accepted. Always check the `accepted` value for each order to know if it succeeded.