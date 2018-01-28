# jsonmsg Specification and Protocol
A simple API specification and protocol alternative to REST (OpenAPI/Swagger), SOAP and json-rpc

jsonmsg is an API specification format and protocol.
It specifies simple, message based communication between servers and clients.
As a central contract, a `spec.json` defines available endpoints with supported protocols, messages with data parameters, and data objects with properties and validation for exchange.

- [Specification (1.0)](blob/master/spec.md)
- [Contract Meta-Schema (1.0)](blob/master/meta.json)

To parse a contract `spec.json` and generate client and server code use:

- [github.com/tfkhsr/jsonmsg](https://github.com/tfkhsr/jsonmsg)
- [github.com/tfkhsr/jsonschema](https://github.com/tfkhsr/jsonschema)

## Status and Compatibility
jsonmsg is developed and currently real-world evaluated at [Stuttgart Media University, Germany](http://www.hdm-stuttgart.de).
At the current state, changes to the specification or meta schema are unlikely so it should be fairly stable.

## Motivation
Modern web applications pose new requirements for communication protocols and specification:
- Fast, bidirectional exchange of data
- Both request/response and message communication pattern
- Stable contracts between clients and servers
- Simple (mental and implementation) transfer of client functionality to API calls
- Lightweight and universal data-interchange formats (typically JSON, Protobuf, ...)
- Support for automatable tooling and inspection

### REST
With the REST architectural style ([Fielding, 2000](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)), these requirements are hard to implement.
It has no concept for bidirectional exchange of data.
Further, it is based on the request/response communication pattern only.
Contracts between clients and server a not a hard requirement and have to be added using third party specification formats such as [OpenAPI/Swagger](https://swagger.io).
The transfer between client functionalities and corresponding API calls can be quite complex when modelling entities with state, non-CRUD actions or patterns such as Command Query Responsibility Segregation ([CQRS, 2011](https://martinfowler.com/bliki/CQRS.html)) are involved.
Further, creating valid API calls requires clients to merge multiple identifiers with action names into URL strings, select correct HTTP methods, set valid HTTP headers (authentication, ...) and encode the HTTP body.
All this has to happen, just so the server can lookup the authentication from HTTP headers, parse URL strings into individual identifiers and action names, select functions based on action names, identifiers and HTTP methods (router) to finally decode the HTTP body if present.

### SOAP
The [SOAP](https://www.w3.org/TR/soap/) messaging protocol provides a remote procedure call style API with [WSDL](https://www.w3.org/TR/2001/NOTE-wsdl-20010315) as specification documents.
It however uses non-lightweight XML, allows the addition of complex extensions and uses the request/response communication pattern.

### json-rpc
[json-rpc](http://www.jsonrpc.org/specification) implements the request/response communcation pattern and has no hard requirement on a contract in the form of an API description.


## Features
To fulfill the requirements of modern web applications, jsonmsg provides a minimal feature set with the following attributes:
- Simple JSON messages with a `msg` name and `data` field:
  - Allow bidirectional exchange of data
  - Support both request/response and message communication pattern
  - Use a lightweight and universal data-interchange format
  - Allow a simple transfer of client functionality to API calls and vice versa
- A JSON Scheme contract `spec.json`:
  - Enforces a defined contract between clients and servers
  - Allows automatic generation of source code, documentation, etc.
  - Defines a transport protocol and language agnostic specification of funcionality


## Examples

On the wire, a message to find a user could look like this:
```
{
  "msg": "findUser",
  "data": {
    "q": "tfkhsr*"
  }
}
```

A corresponding API `spec.json` (a JSON schema in itself) could look like this:
```
{
  "endpoints": {
    "host": "api.jsonmsg.github.io/v1",
    "protocols": ["http", "websocket"],
    "tls": true
  },
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

After sending the `fetchUser` message to an endpoint (via HTTPS or WSS), the client should receive one of two possible data message as defined in the message `outs`.

Either a `user` data message:
```
{
  "msg": "user",
  "data": {
    "id": "tfkhsr",
    "name": "Thomas Fankhauser"
  }
}
```

Or a self defined `error` data message (jsonmsg does not define any error types):
```
{
  "msg": "error",
  "data": {
    "message": "Sorry, no user found"
  }
}
```

Message `in` and `outs` are optional.
This allows the implementation of fire-and-forget scenarios.
