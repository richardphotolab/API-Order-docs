<p align="center">
  <img width="140" height="102" src="https://www.richardphotolab.com/themes/rpl/assets/img/rpl-logo.png">
</p>

# Richard Order API

- [Overview](#overview)
  * [Support](#support)
  * [Anatomy](#anatomy)
- [Authentication](#authentication)
  * [Headers](#headers)
- [Payload](#payload)
- [Responses](#responses)
  * [Service Errors](#service-errors)
  * [Items](#items)
  * [Errors](#errors)

## Overview

The Order API is a private REST interface for approved Richard partners to submit new order registrations for processing.

### Support
For API support, please email api.support@richardphotolab.com

### Anatomy
A valid request is made up of proper http headers, and a payload formatted in valid **JSON**. The payload structure is designed to provide the opportunity to include one or many order registrations, as needed. Details about possible errors and success of each item are included in the response data.

<br/>




## Authentication
Access to the API is granted by providing the proper `Authentication` *HTTP header* containing your token **[JWT](https://jwt.io/introduction/)**.
> :warning: Your unique JWT and the endpoint URL are provided by your Richard technical contact

### Headers

Header | Type | Required | Default | Description
-------|:----:|:--------:|:-------:|------------
`Authorization` | _string_ | Yes | ~ | Your JWT with the Bearer prefix (e.g. `Bearer <JWT>`)
`X-RPL-Mode` | _integer_ (`0`/`1`) | No | `0` | Mode of request live or a test (`0`-test/`1`-live)

<br/>



## Payload

A request payload consists of a base container _array_ containing shipment notification _objects_. There must be a minimum of one(1) item, and a maximum of fifty(50)

### Checksum

All requests require an *HTTP Header* containing a checksum hash generated from the payload contents. This is accomplished by striping the payload of any whitespace (including between words) and pushing it through a simple MD5 hashing mechanism (your developer will be familiar with this).

The whitespace replacement regular expression we recommend (and use ourselves) is simply `\s+` (_multiline_) & (_global_) replaced with nothing `''`.

#### Example

Before replacement:
```json
{
	"Header": {
      "OrderPONum": "PO0061",
      "Customer": {
        "AccountName": "Richard Friend",
        "Phone": 1111111111,
        "Email": "support@domain.com"
      },
      "Shipping": {
        "CustomerName": "Richard Person",
        "Address1": "123 Street Address",
        "Address2": "Apt #6",
        "City": "Big City",
        "State": "CA",
        "PostalCode": "11111",
        "Country": "USA",
        "Phone": 1111111111
      }
  }
}
```

After replacement:
```json
{"Header":{"OrderPONum":"PO0061","Customer":{"AccountName":"RichardFriend","Phone":1111111111,"Email":"support@domain.com"},"Shipping":{"CustomerName":"RichardPerson","Address1":"123StreetAddress","Address2":"Apt#6","City":"BigCity","State":"CA","PostalCode":"11111","Country":"USA","Phone":1111111111}}}
```

> :warning: The replaced version of the payload should only be used to generate the MD5 hash, then discarded

#### Header

Header | Type | Required | Default | Description
-------|:----:|:--------:|:-------:|------------
`RPL-X-PayloadChecksum` | _string_ | Yes | ~ | The request hash (e.g. `9CBE3F5D9A06E74DCFAA371699902A12`)

> :warning: Because this hash is used only for payload verification and not authorization, the known security issues with MD5 are of negligible concern

### Requests

**Method:** `PUT`

_object_

Field | Type | Required | Limits |Description
------|:----:|:--------:|:------:|-----------
`Header` | _object_ | Yes | | Order header
&nbsp;&nbsp;&nbsp;&nbsp;`OrderPONum`                              | _string_  | Yes | max 20  | PO Number
&nbsp;&nbsp;&nbsp;&nbsp;`OrderReference`                          | _string_  | No  | max 50  | Reference ID
&nbsp;&nbsp;&nbsp;&nbsp;`OrderPromoCode`                          | _string_  | No  | max 50  | Promotion Code
&nbsp;&nbsp;&nbsp;&nbsp;`Customer`                                | _object_  | Yes |         | Customer
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`AccountName`     | _string_  | Yes | max 50  | Account Name
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`Phone`           | _integer_ | Yes | max 18  | Account Phone Number
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`Email`           | _string_  | Yes | max 100 | Account Email Address
&nbsp;&nbsp;`Shipping` | _object_ | Yes | | Moo
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`CustomerName`    | _string_  | Yes | max 100 | Shipping Name
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`Address1`        | _string_  | Yes | max 100 | Shipping Address 1
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`Address2`        | _string_  | Yes | max 100 | Shipping Address 2
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`City`            | _string_  | Yes | max 50  | Shipping City
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`State`           | _string_  | Yes | max 50  | Shipping State
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`PostalCode`      | _string_  | Yes | max 16  | Shipping Postal Code
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`Country`         | _string_  | Yes | max 50  | Shipping Country
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`Phone`           | _integer_ | Yes | max 18  | Shipping Phone Number
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`ShippingMethod`  | _string_  | Yes | max 20  | Shipping Method
`Items` | _array_ | Yes | | Items
&nbsp;&nbsp;&nbsp;&nbsp;_(recurring object)_ | _object_ | Yes | | Moo
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`ItemCode`        | _string_  | Yes | max 50  | Item Code
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`Quantity`        | _integer_ | Yes | max 3   | Item Quantity
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`IPath`           | _string_  | No  | URL | Image URL
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`Options`         | _string_  | No  |   | Item Options
`Options` | _array_ | No | | Options
&nbsp;&nbsp;&nbsp;&nbsp;_(recurring object)_ | _object_ | No | Moo
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`ItemCode`        | _string_  | Yes | | Item Code
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`Quantity`        | _integer_ | No  | | Item Quantity
```JSON
[
  {
    "Header": {
      "OrderPONum": "PO0061",
      "Customer": {
        "AccountName": "Richard Partner",
        "Phone": 1111111111,
        "Email": "support@domain.com"
      },
      "Shipping": {
        "CustomerName": "Partner Customer",
        "Address1": "123 Street Address",
        "Address2": "Apt #6",
        "City": "Salt Lake City",
        "State": "Utah",
        "PostalCode": "11111",
        "Country": "USA",
        "Phone": 1111111111,
        "ShippingMethod": "SPSSP"
      }
    },
"Items": [
  {
    "ItemCode": "CP1010P",
    "Quantity": 1,
    "IPath": "http://www.google.co.in/intl/en_com/images/srpr/logo1w.jpg",
    "Options": [
      {
        "ItemCode": "MT1020S2"
      }
    ]
  },
  {
    "ItemCode": "CP1020D",
    "Quantity": 1,
    "IPath": "http://www.google.co.in/intl/en_com/images/srpr/logo1w.jpg",
    "CropDetails": "[0.2, 0.1, 0.95, 0.85]",
    "Orientation": 90
  },
  {
    "ItemCode": "CP1010P",
    "Quantity": 1,
    "IPath": "http://www.google.co.in/intl/en_com/images/srpr/logo1w.jpg",
    "Options": [
      {
        "ItemCode": "MT1020S2"
      }
    ]
  }
],
"Options": [
  {
    "ItemCode": "SPCC",
    "Quantity": 2
  },
  {
    "ItemCode": "SPEP"
  }
]
```
<br/>


## Responses

All responses from the service will be formatted as valid JSON.

The response will include an `errors` array containing any errors which occurred during the processing of your request, and an `items` array containing details of each item in your request (unless a Service Error).

All response payloads, no matter the type, also include a `mode` boolean integer field indicating if the request payload was processed in a test or live mode.

### Service Errors

Service errors are low level failures which block any further processing of the request payload. These include authentication failures as well as malformed payloads (invalid JSON structure, missing required fields). The type of error is determined by examining the return HTTP status code along with the error message(s) in the returned payload.

The payload is made up of an `errors` array containing a message line for each error which occurred.
> :ok_hand:	In most circumstances service errors generate only one message line

_object_
Field | Type | Description
------|:----:|------------
`errors` | _string_ | Message of the error which occurred

#### 401 Unauthorized
HTTP Code: `401`
```JSON
{
  "errors": [ "Authentication failure" ]
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

### Items
_object_
Field | Type | Description
------|:----:|------------
`index` | _integer_ | Internally generated reference for each item
`order_number` | _string_ \| _null_ | Provided order number for this item
`accepted` | _integer_ (`0`/`1`) | Value representing if the update was accepted
`created_at` | _string_ \| _null_ | Full string date and time in UTC when the order was accepted

#### 200 Created
HTTP Code: `200` (Success)
```JSON
{
  "errors": [],
  "items": [
    {
      "index": 0,
      "order_number": "RP9876",
      "accepted": 1,
      "created_at": "2019-11-27T16:26:59.000000Z",
    },
    {
      "index": 1,
      "order_number": "RP9879",
      "accepted": 1,
      "created_at": "2019-11-27T16:26:59.000000Z",
    }
  ]
}
```
<br/>

### Errors

Errors occur at the item level and are reported for each item individually. If a problem occurs with an item, a corresponding error object with matching index will be populated in the errors array.

_object_
Field | Type | Description
------|:----:|------------
`index` | _integer_ | Internal reference for each item (_in the event of a malformed payload_)
`order_number` | _string_ \| _null_ | Provided order number for this item
`messages` | _string_ | Messages for the errors which occurred

> :warning: Item level errors do not modify the return http code.
> 

#### Example

PUT:
```JSON
[
  {
    "order_number": "RP9090",
    "tracking_number": 123456789
  },
  {
    "order_number": "",
    "tracking_number": ""
  }
]
```

HTTP Code: `200`
```JSON
{
  "errors": [
    {
      "index": 0,
      "order_number": "RP9090",
      "messages": [
        "Required field 'tracking_number' is invalid"
      ]
    },
    {
      "index": 1,
      "order_number": null,
      "messages": [
        "Required field 'order_number' is empty",
        "Required field 'tracking_number' is empty"
      ]
    }
  ],
  "items": [
    {
      "index": 0,
      "order_number": "RP9090",
      "accepted": 0,
      "created_at": null,
    },
    {
      "index": 1,
      "order_number": null,
      "accepted": 0,
      "created_at": null,
    }
  ]
}
```