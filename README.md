<p align="center">
  <img width="140" height="102" src="https://www.richardphotolab.com/themes/rpl/assets/img/rpl-logo.png">
</p>

# Richard Order API

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

The Order API is a private REST interface for approved Richard partners to submit new order registrations for processing.

### Support

For API support, please email api.support@richardphotolab.com

### Anatomy

A valid request is made up of proper http headers, and a payload formatted in valid **JSON**. The payload structure is designed to provide the opportunity to include one or many order registrations, as needed. Details about possible errors and success of each item are included in the response data.

<br/>

## Authorization

Access to the API is granted by providing the proper `Authorization` _HTTP header_ containing your token **[JWT](https://jwt.io/introduction/)**.

> :warning: Your unique JWT and the endpoint URL are provided by your Richard technical contact

### Headers

| Header          |        Type         | Required | Default | Description                                           |
| --------------- | :-----------------: | :------: | :-----: | ----------------------------------------------------- |
| `Authorization` |      _string_       |   Yes    |    ~    | Your JWT with the Bearer prefix (e.g. `Bearer <JWT>`) |
| `RPL-X-Mode`    | _integer_ (`0`/`1`) |    No    |   `0`   | Mode of request live or a test (`0`-test/`1`-live)    |

<br/>

## Payload

A request payload consists of a base container _array_ containing shipment notification _objects_. There must be a minimum of one(1) item, and a maximum of fifty(50)

### Checksum

All requests require an _HTTP Header_ containing a checksum hash generated from the payload contents. This is accomplished by striping the payload of any whitespace (including between words) and pushing it through a simple MD5 hashing mechanism (your developer will be familiar with this).

The whitespace replacement regular expression we recommend (and use ourselves) is simply `\s+` (_multiline_) & (_global_) replaced with nothing `''`.

#### Example

Before replacement:

```json
{
  "header": {
    "orderNumber": "PO0061",
    "customer": {
      "accountName": "Richard Friend",
      "phone": 1111111111,
      "email": "support@domain.com"
    },
    "shipping": {
      "customerName": "Richard Person",
      "address1": "123 Street Address",
      "address2": "Apt #6",
      "city": "Big City",
      "state": "CA",
      "postalCode": "11111",
      "country": "USA",
      "phone": 1111111111
    }
  }
}
```

After replacement:

```json
[{"header":{"orderPONum":"PO0061","customer":{"accountName":"RichardPartner","phone":1111111111,"email":"support@domain.com"},"shipping":{"customerName":"PartnerCustomer","address1":"123StreetAddress","address2":"Apt#6","city":"SaltLakeCity","state":"Utah","postalCode":"11111","country":"USA","phone":1111111111,"shippingMethod":"SPSSP"}},"items":[{"itemCode":"CP1010P","quantity":1,"iPath":"http://www.google.co.in/intl/en_com/images/srpr/logo1w.jpg","options":[{"itemCode":"MT1020S2"}]},{"itemCode":"CP1020D","quantity":1,"iPath":"http://www.google.co.in/intl/en_com/images/srpr/logo1w.jpg","cropDetails":"[0.2,0.1,0.95,0.85]","orientation":90},{"itemCode":"CP1010P","quantity":1,"iPath":"http://www.google.co.in/intl/en_com/images/srpr/logo1w.jpg","options":[{"itemCode":"MT1020S2"}]}],"options":[{"itemCode":"SPCC","quantity":2},{"itemCode":"SPEP"}]
```

> :warning: The replaced version of the payload should only be used to generate the MD5 hash, then discarded

#### Header

| Header                  |   Type   | Required | Default | Description                                                |
| ----------------------- | :------: | :------: | :-----: | ---------------------------------------------------------- |
| `RPL-X-PayloadChecksum` | _string_ |   Yes    |    ~    | The request hash (e.g. `9CBE3F5D9A06E74DCFAA371699902A12`) |

> :warning: Because this hash is used only for payload verification and not authorization, the known security issues with MD5 are of negligible concern

### Requests

**Method:** `PUT`

_object_

| Field                                                            |   Type    | Required | Limits  | Description           |
| ---------------------------------------------------------------- | :-------: | :------: | :-----: | --------------------- |
| `header`                                                         | _object_  |   Yes    |         | Order header          |
| &nbsp;&nbsp;&nbsp;&nbsp;`orderNumber`                            | _string_  |   Yes    | max 20  | Order Number          |
| &nbsp;&nbsp;&nbsp;&nbsp;`uniqueId`                               | _string_  |    No    | max 80  | Unique Identifier     |
| &nbsp;&nbsp;&nbsp;&nbsp;`orderReference`                         | _string_  |    No    | max 50  | Reference ID          |
| &nbsp;&nbsp;&nbsp;&nbsp;`orderPromoCode`                         | _string_  |    No    | max 50  | Promotion Code        |
| &nbsp;&nbsp;&nbsp;&nbsp;`customer`                               | _object_  |   Yes    |         | Customer              |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`accountName`    | _string_  |   Yes    | max 50  | Account Name          |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`phone`          | _integer_ |   Yes    | max 18  | Account Phone Number  |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`email`          | _string_  |   Yes    | max 100 | Account Email Address |
| &nbsp;&nbsp;`shipping`                                           | _object_  |   Yes    |         | Moo                   |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`customerName`   | _string_  |   Yes    | max 100 | Shipping Name         |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`address1`       | _string_  |   Yes    | max 100 | Shipping Address 1    |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`address2`       | _string_  |   Yes    | max 100 | Shipping Address 2    |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`city`           | _string_  |   Yes    | max 50  | Shipping City         |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`state`          | _string_  |   Yes    | max 50  | Shipping State        |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`postalCode`     | _string_  |   Yes    | max 16  | Shipping Postal Code  |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`country`        | _string_  |   Yes    | max 50  | Shipping Country      |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`phone`          | _integer_ |   Yes    | max 18  | Shipping Phone Number |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`shippingMethod` | _string_  |   Yes    | max 20  | Shipping Method       |
| `items`                                                          |  _array_  |   Yes    |         | Items                 |
| &nbsp;&nbsp;&nbsp;&nbsp;_(recurring object)_                     | _object_  |   Yes    |         |                       |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`itemCode`       | _string_  |   Yes    | max 50  | Item Code             |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`quantity`       | _integer_ |   Yes    |  max 3  | Item Quantity         |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`iPath`          | _string_  |    No    |   URL   | Image URL             |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`options`        | _string_  |    No    |         | Item Options          |
| `options`                                                        |  _array_  |    No    |         | Options               |
| &nbsp;&nbsp;&nbsp;&nbsp;_(recurring object)_                     | _object_  |    No    |   Moo   |						|
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`itemCode`       | _string_  |   Yes    |         | Item Code             |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`quantity`       | _integer_ |    No    |         | Item Quantity         |

```JSON
[
  	{
		"header": {
		"orderPoNum": "PO0061",
		"customer": {
			"accountName": "Richard Partner",
			"phone": 1111111111,
			"email": "support@domain.com"
		},
		"shipping": {
			"customerName": "Partner Customer",
			"address1": "123 Street Address",
			"address2": "Apt #6",
			"city": "Salt Lake City",
			"state": "Utah",
			"postalCode": "11111",
			"country": "USA",
			"phone": 1111111111,
			"shippingMethod": "SPSSP"
		}
		},
		"items": [
			{
				"itemCode": "CP1010P",
				"quantity": 1,
				"iPath": "http://www.google.co.in/intl/en_com/images/srpr/logo1w.jpg",
				"options": [
					{
						"itemCode": "MT1020S2"
					}
				]
			},
			{
				"itemCode": "CP1020D",
				"quantity": 1,
				"iPath": "http://www.google.co.in/intl/en_com/images/srpr/logo1w.jpg",
				"cropDetails": "[0.2, 0.1, 0.95, 0.85]",
				"orientation": 90
			},
			{
				"itemCode": "CP1010P",
				"quantity": 1,
				"iPath": "http://www.google.co.in/intl/en_com/images/srpr/logo1w.jpg",
				"options": [
					{
						"itemCode": "MT1020S2"
					}
				]
			}
		],
		"options": [
			{
				"itemCode": "SPCC",
				"quantity": 2
			},
			{
				"itemCode": "SPEP"
			}
		]
  	}
]
```

<br/>

## Responses

All responses from the service will be formatted as valid JSON.

The response will include an `errors` array containing any errors which occurred during the processing of your request, and an `items` array containing details of each item in your request (unless a Service Error).

All response payloads except Service Errors (see below), also include a `mode` boolean integer field indicating if the request payload was processed in a test or live mode.

### Service Errors

Service errors are low level failures which block any further processing of the request payload. These include authorization failures as well as malformed payloads (invalid JSON structure, missing required fields). The type of error is determined by examining the return HTTP status code along with the error message(s) in the returned payload.

The payload is made up of an `errors` array containing a message line for each error which occurred.

> :ok_hand: In most circumstances service errors generate only one message line

_object_
Field | Type | Description
------|:----:|------------
`errors` | _array_[_string_] | Messages of the error which occurred

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
`order_number` | _string_ \| _null_ | Provided order number for this item
`accepted` | _integer_ (`0`/`1`) | Value representing if the order was accepted
`created_at` | _string_ \| _null_ | Full string date and time in UTC when the order was accepted

#### 200 Created

HTTP Code: `200` (Success)

```JSON
{
	"errors": [],
	"orders": [
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
	"orders": [
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
