# JSON-RPC X

A new specification developed to implement method nesting of remote procedure calls

* Time of origin: 2022-06-10
* Update: 2022-06-10
* Author: luxuncang `<luxuncang@qq.com>`

[简体中文](./README.zh.md) | English

## Summary

`Json-rpc x` is a stateless and lightweight remote procedure call (RPC) protocol. This specification mainly defines some data structures and related processing rules. It allows to run in the same process based on socket, HTTP and many other different message transmission environments. It is an extension of `json-rpc 2.0`, including nested calls and object instantiation.

## Appointment

Keywords in document `"MUST"`、`"MUST NOT"`、`"REQUIRED"`、`"SHALL"`、`"SHALL NOT"`、`"SHOULD"`、`"SHOULD NOT"`、`"RECOMMENDED"`、`"MAY"` and `"OPTIONAL"` in [RFC 2119](http://www.ietf.org/rfc/rfc2119.txt) get detailed explanation and description in.

Because `JSON RPC` uses `JSON`, it has the same type system as `JSON` (see[http://www.json.org](http://www.json.org/) or [RFC 4627](http://www.ietf.org/rfc/rfc4627.txt))。`JSON` can represent four basic types(`String`、`Numbers`、`Booleans` and `Null`)and two structured types(Objects和Arrays)。In the specification, terms `"Primitive"` Mark the 4 original types，`"Structured"`Two structural types are marked. Whenever a document involves JSON data types, the first letter must be capitalized：`Object`，`Array`，`String`，`Number`，`Boolean`，`Null`.Include `True` and `False`.

All member names exchanged between the client and any matched server should be case sensitive. Functions, methods, and procedures can be considered interchangeable.

The client is defined as the source of the request object and the handler of the response object.

The server is defined as the origin of the response object and the handler of the request object.

One implementation of the specification is that it can easily fill these two roles, even at the same time, the same client or other different clients. This specification does not cover complex layers.

## Compatibility

`The request object and response object of `JSON-RPC X` may not work completely on the existing `json-rpc 1.0` and `json-rpc 1.0` client or server. However, we can easily distinguish "X" among the three versions, and there will always be a member named 'jsonrpc' with a value of 'x'. Most x implementations should consider trying to handle `1.0` and `2.0` objects. Even if they are not peer-to-peer, they should also be prompted.

## Request object

Sending a request object to the server represents an RPC call. A request object contains the following members:

**jsonrpc**

> The string specifying the JSON RPC Protocol version must be exactly written as `"X"`

**method**

> The string list `list[str]` containing the name of the method to be called. The method name starting with RPC and connected with English period (u+002e or ASCII 46) are the method name and extension reserved for RPC, and cannot be used elsewhere.

**params**

> The list of structural parameter values required for calling the method `list[none | list | dict]`. The member parameters can be `none`, `{}`, `[]`. When the value is' list 'or' dict ', the object is' callable'. When the value is `None`, the opposite is true.

**id**

> The unique identification ID of the established client. The value must contain a string, numeric value or null. If the member is not included, it is deemed to be a notice. This value is generally not null. If it is a numeric value, it should not contain decimals.

The server must answer the same value if it is included in the response object. This member is used for the context of association between two objects.

### notice

The request object that does not contain the `"Id"` member is a notification. As a notification request object, the client is not interested in the corresponding response object, and there is no response object to return to the client. The server must not reply to a notification, including those in batch requests.

Since the notification has no response object returned, it is uncertain whether the notification is defined. Similarly, the client is not aware of any errors (such as parameter defaults, internal errors).

### Parameter structure

The RPC call must be a parameter value of a basic type or a structured type if there are parameters, either an indexed array or an associative array object.

* Index: the parameter must be an array and contain parameter values consistent with the expected order of the server.
* Association name: the parameter must be an object and contain the parameter member name matching the server. A member name that is not expected may cause an error. The name must match exactly, including the expected parameter name and case of the method.

## Response object

When an RPC call is initiated, the server must respond except for the notification. The response is represented as a JSON object, using the following members:

**jsonrpc**

> The string specifying the JSON RPC Protocol version must be exactly written as `"X"`

**result**

> The member must contain when successful.
>
> The member must not be included when a method call causes an error.
>
> The called method in the server determines the value of the member.

**error**

>The member must be included when it fails.
>
>The member must not be included when no error is caused.
>
>The member parameter value must be the object defined in the `'error object'`.

**id**

>The member must contain.
>
>The member value must be consistent with the ID member value in the request object.
>
>If there is an error when checking the request object ID (such as parameter error or invalid request), the value must be null.
The response object must contain `'result'` or `'error'` members, but the two members must not contain both.

### Error Object

When an RPC call encounters an error, the returned response object must contain the error member parameters and be an object with the following member parameters:

**code**

> Use a numeric value to indicate the error type of the exception. Must be an integer.

**message**

> A simple description string for the error. The description should be limited to a short sentence as far as possible.

**data**

> A basic or structured type that contains additional information about the error. This member can be ignored. The member value is defined by the server (such as detailed error information, nested errors, etc).

-32768 to -32000 are reserved predefined error codes. Error codes within this range cannot be clearly defined and the following are reserved for future use. The error code is basically the same as that suggested by XML RPC，url： [http://xmlrpc-epi.sourceforge.net/specs/rfc.fault_codes.php](http://xmlrpc-epi.sourceforge.net/specs/rfc.fault_codes.php)

| code             | message                    | meaning                                                    |
| ---------------- | -------------------------- | ---------------------------------------------------------- |
| -32700           | Parse error   | The server received an invalid JSON. This error is sent when the server attempts to parse JSON text |
| -32600           | Invalid Request    | The JSON sent is not a valid request object.                         |
| -32601           | Method not found | The method does not exist or is invalid.                                  |
| -32602           | Invalid params   | Invalid method parameter.                                       |
| -32603           | Internal error     | JSON RPC internal error.                                       |
| -32000 to -32099 | Server error     | Reserved server errors for customization.                         |

In addition, the remaining error type codes are available to the application as custom errors.

## Batch Call

When multiple request objects need to be sent at the same time, the client can send an array containing all the request objects.

When all the request objects of the batch call are processed, the server needs to return an array containing the corresponding response objects. Each response object should respond to each request object, unless it is a request object for notification. The server can handle these batch calls concurrently in any order and with any width of parallelism.

These corresponding response objects can be included in the returned array in any order, and the client should match the corresponding request object based on the ID members in each response object.

If the RPC operation called in batch is not a valid JSON or an array containing at least one value, the server will return only a response object instead of an array. If the batch call has no response object to return, the server does not need to return any results and must not return an empty array to the client.

> `Batch Call` is not recommended in `JSON-RPC X`.

## Example

Syntax:

```
--> data sent to Server
<-- data sent to Client
```

RPC call with indexed array parameters:

```python
def subtract(minuend, subtrahend):
    return minuend - subtrahend

subtract(42, 23)
--> {"jsonrpc": "X", "method": ["subtract"], "params": [[42, 23]], "id": 1}
<-- {"jsonrpc": "X", "result": 19, "id": 1}

subtract(23, 42)
--> {"jsonrpc": "X", "method": ["subtract"], "params": [[23, 42]], "id": 2}
<-- {"jsonrpc": "X", "result": -19, "id": 2}
```

RPC call with associative array parameters:

```python
def subtract(minuend, subtrahend):
    return minuend - subtrahend

subtract(subtrahend = 23, minuend = 42)
--> {"jsonrpc": "X", "method": ["subtract"], "params": [{"subtrahend": 23, "minuend": 42}], "id": 3}
<-- {"jsonrpc": "X", "result": 19, "id": 3}

subtract(minuend = 42, subtrahend = 23)
--> {"jsonrpc": "X", "method": ["subtract"], "params": {["minuend": 42, "subtrahend"]: 23}, "id": 4}
<-- {"jsonrpc": "X", "result": 19, "id": 4}
```

RPC calls for chained calls:

```python
class Math:

    @staticmethod
    def subtract(minuend, subtrahend):
        return minuend - subtrahend

Math.subtract(23, 42)
--> {"jsonrpc": "X", "method": ["subtract", "add"], "params": [null, [23, 42]], "id": 5}
<-- {"jsonrpc": "X", "result": -19, "id": 5}

Math.subtract(minuend = 23, subtrahend = 42)
--> {"jsonrpc": "X", "method": ["subtract", "add"], "params": [null, {"minuend": 23, "subtrahend": 42}], "id": 6}
<-- {"jsonrpc": "X", "result": -19, "id": 6}
```

RPC calls with chained calls to generate instances:

```python
class Math:

    def __init__(self, minuend):
        self.minuend = minuend
  
    def add(self, addend):
        self.minuend = self.minuend + addend
        return self

    def subtract(self, subtrahend):
        self.minuend = self.minuend - subtrahend
        return self


Math(10).add(20).subtract(30).minuend
--> {"jsonrpc": "X", "method": ["Math", "add", "subtract", "minuend"], "params": [10, [20], [30], null], "id": 5}
<-- {"jsonrpc": "X", "result": 0, "id": 5}
```

Notice:

```python
--> {"jsonrpc": "X", "method": ["update"], "params": [1,2,3,4,5]}
--> {"jsonrpc": "X", "method": ["foobar"]}
```

RPC call without calling method:

```python
--> {"jsonrpc": "X", "method": ["foobar"], "id": "1"}
<-- {"jsonrpc": "X", "error": {"code": -32601, "message": "Method not found"}, "id": "1"}
```

RPC call with invalid JSON:

```python
--> {"jsonrpc": "X", "method": "foobar, "params": "bar", "baz]
<-- {"jsonrpc": "X", "error": {"code": -32700, "message": "Parse error"}, "id": null}
```

RPC call with invalid request object:

```python
--> {"jsonrpc": "X", "method": 1, "params": ["bar"]}
<-- {"jsonrpc": "X", "error": {"code": -32600, "message": "Invalid Request"}, "id": null}
```

RPC batch call with invalid JSON:

```python
--> [
        {"jsonrpc": "X", "method": ["sum"], "params": [1,2,4], "id": "1"},
        {"jsonrpc": "X", "method"
    ]
<-- {"jsonrpc": "X", "error": {"code": -32700, "message": "Parse error"}, "id": null}
```

RPC call with empty array:

```python
--> []
<-- {"jsonrpc": "X", "error": {"code": -32600, "message": "Invalid Request"}, "id": null}
```

Non empty and invalid RPC batch call:

```python
--> [1]
<-- [
    {"jsonrpc": "X", "error": {"code": -32600, "message": "Invalid Request"}, "id": null}
    ]
```

Invalid RPC bulk call:

```python
--> [1,2,3]
<-- [
    {"jsonrpc": "X", "error": {"code": -32600, "message": "Invalid Request"}, "id": null},
    {"jsonrpc": "X", "error": {"code": -32600, "message": "Invalid Request"}, "id": null},
    {"jsonrpc": "X", "error": {"code": -32600, "message": "Invalid Request"}, "id": null}
    ]
```

RPC batch call:

```python
--> [
    {"jsonrpc": "X", "method": ["sum"], "params": [[1,2,4]], "id": "1"},
    {"jsonrpc": "X", "method": ["notify_hello"], "params": [[7]]},
    {"jsonrpc": "X", "method": ["subtract"], "params": [[42,23]], "id": "2"},
    {"foo": "boo"},
    {"jsonrpc": "X", "method": ["foo", "get"], "params": [null, {"name": "myself"}], "id": "5"},
    {"jsonrpc": "X", "method": ["get_data"], "id": "9"}
    ]
<-- [
    {"jsonrpc": "X", "result": 7, "id": "1"},
    {"jsonrpc": "X", "result": 19, "id": "2"},
    {"jsonrpc": "X", "error": {"code": -32600, "message": "Invalid Request"}, "id": null},
    {"jsonrpc": "X", "error": {"code": -32601, "message": "Method not found"}, "id": "5"},
    {"jsonrpc": "X", "result": ["hello", 5], "id": "9"}
    ]
```

All RPC bulk calls that are notifications:

```python
--> [
    {"jsonrpc": "X", "method": ["notify_sum"], "params": [1,2,4]},
    {"jsonrpc": "X", "method": ["notify_hello"], "params": [7]}
]

<-- //Nothing is returned for all notification batches
```

## 扩展

Method names starting with RPC are reserved as system extensions and must not be used elsewhere. Each system extension shall have relevant specification documents, and all system extensions shall be optional.

Copyright (C) 2007-2010 by the JSON-RPC Working Group

This document and translations of it may be used to implement JSON-RPC, it may be copied and furnished to others, and derivative works that comment on or otherwise explain it or assist in its implementation may be prepared, copied, published and distributed, in whole or in part, without restriction of any kind, provided that the above copyright notice and this paragraph are included on all such copies and derivative works. However, this document itself may not bemodified in any way.

---

The text on this page allows you to [CC-BY-SA-3](http://zh.wikipedia.org/wiki/Wikipedia:CC-BY-SA-3.0%E5%8D%8F%E8%AE%AE%E6%96%87%E6%9C%AC)和[GNU Free Document license](http://zh.wikipedia.org/wiki/Wikipedia:GNU%E8%87%AA%E7%94%B1%E6%96%87%E6%A1%A3%E8%AE%B8%E5%8F%AF%E8%AF%81%E6%96%87%E6%9C%AC)下修改和再使用
