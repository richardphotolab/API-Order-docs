<p align="center">
  <img width="140" height="102" src="https://www.richardphotolab.com/themes/rpl/assets/img/rpl-logo.png">
</p>

# Response

All responses from endpoints will be formatted as valid _JSON_.

The response will include an `errors` array containing any errors which occurred during the processing of your request, and an `orders` array containing details of each item in your request (unless a Service Error).

Response payloads except Service Errors (see below), may also include a `mode` Boolean integer field (if the endpoint supports it) indicating if the request payload was processed in a _test_ or _live_ mode.

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