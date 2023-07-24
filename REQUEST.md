<p align="center">
  <img width="140" height="102" src="https://gfs-na.richardphotolab.com/img/logo/rpl-logo.png">
</p>

# Request


### Connections

All connections are made using HTTP/1.1 or above, on port 443 with SSL.

> :warning: The required endpoint addresses will be provided by your Richard technical contact.

### Anatomy

A valid request is made up of proper _http headers_, and a payload formatted as valid **JSON**. The payload structure is dependent on which endpoint you are using; see individual endpoint documentation for details.


### Authentication

You can authenticate your connection by providing the proper `Authorization` _HTTP header_ containing your token.

> :warning: Your token will be provided by your Richard technical contact.

| Header          |        Type         | Required | Default | Description                                           |
| --------------- | :-----------------: | :------: | :-----: | ----------------------------------------------------- |
| `authorization` |      _string_       |   Yes    |    ~    | Your token with the Bearer prefix (e.g. `Bearer <token>`) |

### Checksum

All requests containing a payload require an _HTTP Header_ containing a checksum hash generated from the payload contents. This is accomplished by striping the payload of any whitespace (including between words) and pushing it through a simple MD5 hashing mechanism (your developer will be familiar with this).

The whitespace replacement regular expression we recommend (and use ourselves) is simply `\s+` (_multiline_) & (_global_) replaced with nothing `''`.

#### Example

Before replacement:

```json
[
  {
    "header": {
      "orderNumber": "PO0061",
      "shipping": {
        "customerName": "Richard Person",
        "address1": "123 Street Address",
        "address2": "Apt #6",
        "city": "Big City",
        "province": "CA",
        "postalCode": "11111",
        "country": "USA",
        "phone": 1111111111
      }
    }
  }
]
```

After replacement:

```json
[{"header":{"orderNumber":"PO0061","shipping":{"customerName":"RichardPerson","address1":"123StreetAddress","address2":"Apt#6","city":"BigCity","province":"CA","postalCode":"11111","country":"USA","phone":1111111111}}}]
```
> :warning: The replaced version of the payload should only be used to generate the MD5 hash, then discarded


#### Header

| Header                  |   Type   | Required | Default | Description                                                |
| ----------------------- | :------: | :------: | :-----: | ---------------------------------------------------------- |
| `rpl-x-payloadchecksum` | _string_ |   Yes    |    ~    | The request hash (e.g. `5e459d27c2bf56bf39adc9a968c59096`) |

> :warning: Because this hash is used only for payload verification and not authorization, the known security issues with MD5 are of no concern

## Note
In times of difficulty, the endpoint will return a response payload containing  error messages which occurred. Make sure you do not discard this response, as it is extremely helpful in debugging concerns.
