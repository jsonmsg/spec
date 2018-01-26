# jsonmsg spec
A simple API specification alternative to REST (OpenAPI/Swagger), SOAP and json-rpc

- [Spec Definition](SPEC.md)
- [Spec Meta-Schema](meta.json)
- [Spec Parser and Code Generator](https://github.com/tfkhsr/jsonmsg)
- [JSON Schema Parser and Code Generator](https://github.com/tfkhsr/jsonschema)

## TL;DR
- REST is too complicated
- SOAP is too complex and XML
- json-rpc is cool, but supports only request/response paradigms
- OpenAPI/Swagger specifies only REST APIs

Therefore, jsonmsg is a message oriented and JSON Schema defined specification format for APIs.
It's certainly not a new idea, but tries to be as simple as possible.

## Motivation
Web applications are getting more and more complex.
Nowadays, data for web applications is typically exchanged via JSON APIs asynchronously in a dynamic fashion.

Traditional design paradigms, such as REST ([Fielding, 2000](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)), identify web resources by their URLs and manipulate them using HTTP methods.
However, modelling resources and their respective collections gets complex as soon as non-CRUD actions need to be performed.
A good example for this is an `Order` that can move through different states, e.g. `created`, `accepted`, `payed`, `rejected`, etc.

Further, the mapping of a request `R` to a function `F` involves complex inspection of many request attributes:

```
route(R_method, R_path, R_headers, R_body) -> F_x(parameters, ...)
```

It is typical for a `route` function to split up a `R_path` string such as `/devices/87g-sdf/locations/298gku-sdf/timespans/asdg-asd` into a `deviceID: 87g-sdf`, `locationID: 298gku-sdf` and a `timespanID: asdg-asd`.
If `R_body` contains data, further parameters need to be extracted from it.
Additionally, important parameters, e.g. for authorization might be delivered via `R_headers` so parameters need to be extracted from them as well.
On the client side, all this information needs to be compiled together, e.g. using string concatination for the path, setting headers and compiling the body.

A possible solution is to use Remote Procedure Call protocols such as [SOAP](https://www.w3.org/TR/soap/) or [json-rpc](http://www.jsonrpc.org/specification).
SOAP, however, has a complex feature set with headers and footers packed into an envelope using XML.
json-rpc relies on a request/response paradigm and thereby is bound to protocols that support such paradigms.

Finally, all paradigms do not support specification driven development out of the box.
Specification frameworks such as [OpenAPI/Swagger](https://swagger.io) only support REST style APIs with a request/response paradigm.


## Goals

The goal for the jsonmsg spec is to provide a minimal feature set with the following attributes:

- The API specification is the central part for development
- API defines messages with names that are exchanged
- Request/Response style is optional
- HTTP methods, paths and headers are used in a minimal fashion
- All parameters needed for routing are embedded in a single data entity
- Definitions does not rely on HTTP request/response protocol (support for both HTTP/Websockets)
- Data definition is based on JSON Schema
- Simple, curlable protocol

The mapping of an incoming request or message therefore only depends on the request body or message:

```
route(R_body) -> F_x(parameters, ..)
route(M) -> F_x(parameters, ..)
```

## Examples

A message to find a user looks like this:
```
{
  "msg": "findUser",
  "data": {
    "q": "tfkhsr*"
  }
}
```
The corresponding API spec (a JSON schema) for this message could look like this:
```
{
  ...
  "messages": {
    "findUser": {
      "in": "#/definitions/query",
      "outs": [
        "#/definitions/user",
        "#/definitions/error"
      ]
    }
  },
  "definitions": {
    "query": {
      "type": "object",
      "properties": {
        "q": { "type": "string" }
      }
    },
    "user": {
      "type": "object",
      "properties": {
        "id": { "type": "string" }
        "name": { "type": "string" }
      }
    },
    "error": {
      "type": "object",
      "properties": {
        "message": { "type": "string" }
      }
    }
  }
}
```
Therefore, one could receive a message with a user:
```
{
  "msg": "user",
  "data": {
    "id": "tfkhsr",
    "name": "Thomas Fankhauser"
  }
}
```
Or an error:
```
{
  "msg": "error",
  "data": {
    "message": "Sorry, no user found"
  }
}
```

The design of errors, data entities and messages is completely up to the developer.
Messages can be sent over any channel:

- An HTTP POST to a single API endpoint
- Via a WebSocket connection
- Via message queue
