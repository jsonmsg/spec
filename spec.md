# jsonmsg Specification 1.0

## Terminology
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

## Overview
APIs MUST be specified using a `spec.json` contract that MUST validate against the jsonmsg [meta-schema](meta.json).
The `spec.json` itself is a JSON Schema as defined in [json-schema-core](http://json-schema.org/latest/json-schema-core.html).

The `spec.json` MUST be served via HTTP(S) to clients at the "endpoints.http" path.
This helps clients to inspect API functionality and generate source code.
For example, if the "endpoints.http" URL is `https://jsonmsg.github.com`, the spec must be served at `https://jsonmsg.github.com/spec.json`.


## Endpoints
The "endpoints" section in the `spec.json` defines supported protocols with corresponding endpoint URLs.

An endpoint key SHOULD be of the following list:

- **http**: HTTP or HTTPS communication as defined in [RFC 2616](https://tools.ietf.org/html/rfc2616)
- **websocket**: WebSocket communication as defined in [RFC 6455](https://tools.ietf.org/html/rfc6455)

An endpoint value MUST define an URI as defined in [RFC 3986](https://tools.ietf.org/html/rfc3986).
The final endpoint URL for a protocol MUST be suffixed with the protocol name to prevent name clashes.
For example, if the "endpoints.http" is `https://jsonmsg.github.com`, the final endpoint URL is `https://jsonmsg.github.com/http`.

### HTTP Protocol Implementation
An HTTP(s) implementation MUST use the `POST` method for every request (except when serving the `spec.json`).
The follwing status codes MUST be used:

| Status Code | Description |
|---|---|
| 200 | Message delivered successfully. Response might contain data message, e.g. `error`. |
| 405 | Invalid method. Only `POST` is allowed. |
| 422 | Unparsable message or invalid format, e.g. missing a `msg` key. |
| 500 | Unrecoverable internal implementation error. |

### WebSocket Protocol Implementation
A WebSocket implementation SHOULD send messages as text message.
Implementations SHOULD be enabled to receive data messages at all times to enable bidirectional communication.

## Messages
The "messages" section defines communication objects where the property name defines the message name and the value defines the message properties.
Message names SHOULD be in CamelCase and MUST only contain alphabet letters `[A-Za-z]`.

A valid message document MUST contain exactly two properties:

| Property | Description |
|---|---|
| msg | The message name as defined in the property name |
| data | Data conforming to either the referenced "in", or one of the "outs" schemas |

For example, a valid message can be:
```
{
  "msg": "fetchUser",
  "data": { ... }
}
```

### Title
The "title" property defines a short description of the message.

### Description
The "description" property defines a long description of the message.

### In
If present, the "in" property MUST be a URI Reference to a JSON Schema.
The referenced schema defines the structure for the "data" property of a message.

If the property is absent, the message MUST not contain a "data" property.

### Outs
If present, the entries in the "outs" array MUST be URI References to JSON Schemas.
The "data" property of a message responding to the initial message MUST conform to one of the referenced schemas.

If the property is absent, the initial message SHOULD not produce follow up messages.

### Group
The "group" property defines a name to combine messages into groups.

## Definitions
The "definitions" section defines data objects that are referenced by messages.
The contents are defined as specified in [json-schema-core](http://json-schema.org/latest/json-schema-core.html).

### Data Messages
Each definition in the "definitions" section also defines a message name used to send data messages.

For example, the definition:
```
{
  ...
  "definitions": {
    "user": {
      "type": "object",
      "properties": {
        "id": { "type": "string" },
        "name": { "type": "string" }
       }
    }
  }
}
```
generates a data message in the form:
```
{
  "msg": "user",
  "data": {
    "id": "...",
    "name": "..."
  }
}
```
