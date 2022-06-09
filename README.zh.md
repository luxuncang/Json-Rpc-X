# JSON-RPC X

为了实现远程过程调用的方法嵌套而拓展的新规范.

* 起源时间: 2022-06-10
* 更新: 2022-06-10
* 作者: luxuncang `<luxuncang@qq.com>`

简体中文 | [English](./README.md)

## 概述

`JSON-RPC X`是一个无状态且轻量级的远程过程调用(RPC)协议。本规范主要定义了一些数据结构及其相关的处理规则。它允许运行在基于socket，http等诸多不同消息传输环境的同一进程中。是 `JSON-RPC 2.0` 的一个拓展，包括嵌套调用和对象实例化。

## 约定

文档中关键字 `"MUST"`、`"MUST NOT"`、`"REQUIRED"`、`"SHALL"`、`"SHALL NOT"`、`"SHOULD"`、`"SHOULD NOT"`、`"RECOMMENDED"`、`"MAY"`和 `"OPTIONAL"` 将在[RFC 2119](http://www.ietf.org/rfc/rfc2119.txt) 中得到详细的解释及描述。

由于JSON-RPC使用JSON，它具有与其相同的类型系统(见[http://www.json.org](http://www.json.org/)或[RFC 4627](http://www.ietf.org/rfc/rfc4627.txt))。JSON可以表示四个基本类型(String、Numbers、Booleans和Null)和两个结构化类型(Objects和Arrays)。规范中，术语 `"Primitive"` 标记那4种原始类型，“Structured”标记两种结构化类型。任何时候文档涉及JSON数据类型，第一个字母都必须大写：`Object`，`Array`，`String`，`Number`，`Boolean`，`Null`。包括 `True` 和 `False` 也要大写。

在客户端与任何被匹配到的服务端之间交换的所有成员名字应是区分大小写的。函数、方法、过程都可以认为是可以互换的。

客户端被定义为请求对象的来源及响应对象的处理程序。

服务端被定义为响应对象的起源和请求对象的处理程序。

该规范的一种实现为可以轻而易举的填补这两个角色,即使是在同一时间,同一客户端或其他不相同的客户端。该规范不涉及复杂层。

## 兼容性

`JSON-RPC X ` 的请求对象和响应对象可能无法完全在现用的 `JSON-RPC 1.0`  和  `JSON-RPC 1.0` 客户端或服务端工作，然而我们可以很容易在三个版本间区分出“X”，总会包含一个成员命名为 `"jsonrpc"` 且值为 `"X"` 。大部分的X实现应该考虑尝试处理 `1.0`和 `2.0`的对象，即使不是对等的也应给其相关提示。

## 请求对象

发送一个请求对象至服务端代表一个rpc调用, 一个请求对象包含下列成员：

**jsonrpc**

> 指定JSON-RPC协议版本的字符串，必须准确写为“X”

**method**

> 包含所要调用方法名称的字符串列表 `List[str]`，其中以rpc开头的方法名，用英文句号（U+002E or ASCII 46）连接的为预留给rpc内部的方法名及扩展名，且不能在其他地方使用。

**params**

> 调用方法所需要的结构化参数值的列表 `List[None | List | Dict]`，该成员参数可以为 `None`, `{}`, `[]`。当值为 `List` 或 `Dict` 时，表示对象是 `Callable`,而当值为 `None` 时恰好相反。

**id**

> 已建立客户端的唯一标识id，值必须包含一个字符串、数值或NULL空值。如果不包含该成员则被认定为是一个通知。该值一般不为NULL，若为数值则不应该包含小数。

服务端必须回答相同的值如果包含在响应对象。 这个成员用来两个对象之间的关联上下文。

### 通知

没有包含 `"id"` 成员的请求对象为通知，作为通知的请求对象表明客户端对相应的响应对象并不感兴趣，本身也没有响应对象需要返回给客户端。服务端必须不回复一个通知，包含那些批量请求中的。

由于通知没有返回的响应对象，所以通知不确定是否被定义。同样，客户端不会意识到任何错误（例如参数缺省，内部错误）。

### 参数结构

rpc调用如果存在参数则必须为基本类型或结构化类型的参数值，要么为索引数组，要么为关联数组对象。

* 索引：参数必须为数组，并包含与服务端预期顺序一致的参数值。
* 关联名称：参数必须为对象，并包含与服务端相匹配的参数成员名称。没有在预期中的成员名称可能会引起错误。名称必须完全匹配，包括方法的预期参数名以及大小写。

## 响应对象

当发起一个rpc调用时，除通知之外，服务端都必须回复响应。响应表示为一个JSON对象，使用以下成员：

**jsonrpc**

> 指定JSON-RPC协议版本的字符串，必须准确写为“X”

**result**

> 该成员在成功时必须包含。
>
> 当调用方法引起错误时必须不包含该成员。
>
> 服务端中的被调用方法决定了该成员的值。

**error**

> 该成员在失败是必须包含。
>
> 当没有引起错误的时必须不包含该成员。
>
> 该成员参数值必须为 `错误对象` 中定义的对象。

**id**

> 该成员必须包含。
>
> 该成员值必须于请求对象中的id成员值一致。
>
> 若在检查请求对象id时错误（例如参数错误或无效请求），则该值必须为空值。

响应对象必须包含 `result` 或 `error` 成员，但两个成员必须不能同时包含。

### 错误对象

当一个rpc调用遇到错误时，返回的响应对象必须包含错误成员参数，并且为带有下列成员参数的对象：

**code**

> 使用数值表示该异常的错误类型。 必须为整数。

**message**

> 对该错误的简单描述字符串。 该描述应尽量限定在简短的一句话。

**data**

> 包含关于错误附加信息的基本类型或结构化类型。该成员可忽略。 该成员值由服务端定义（例如详细的错误信息，嵌套的错误等）。

-32768至-32000为保留的预定义错误代码。在该范围内的错误代码不能被明确定义，保留下列以供将来使用。错误代码基本与XML-RPC建议的一样，url： [http://xmlrpc-epi.sourceforge.net/specs/rfc.fault_codes.php](http://xmlrpc-epi.sourceforge.net/specs/rfc.fault_codes.php)

| code             | message                    | meaning                                                    |
| ---------------- | -------------------------- | ---------------------------------------------------------- |
| -32700           | Parse error语法解析错误    | 服务端接收到无效的json。该错误发送于服务器尝试解析json文本 |
| -32600           | Invalid Request无效请求    | 发送的json不是一个有效的请求对象。                         |
| -32601           | Method not found找不到方法 | 该方法不存在或无效                                         |
| -32602           | Invalid params无效的参数   | 无效的方法参数。                                           |
| -32603           | Internal error内部错误     | JSON-RPC内部错误。                                         |
| -32000 to -32099 | Server error服务端错误     | 预留用于自定义的服务器错误。                               |

除此之外剩余的错误类型代码可供应用程序作为自定义错误。

## 批量调用

当需要同时发送多个请求对象时，客户端可以发送一个包含所有请求对象的数组。

当批量调用的所有请求对象处理完成时，服务端则需要返回一个包含相对应的响应对象数组。每个响应对象都应对应每个请求对象，除非是通知的请求对象。服务端可以并发的，以任意顺序和任意宽度的并行性来处理这些批量调用。

这些相应的响应对象可以任意顺序的包含在返回的数组中，而客户端应该是基于各个响应对象中的id成员来匹配对应的请求对象。

若批量调用的rpc操作本身非一个有效json或一个至少包含一个值的数组，则服务端返回的将单单是一个响应对象而非数组。若批量调用没有需要返回的响应对象，则服务端不需要返回任何结果且必须不能返回一个空数组给客户端。

> 在 `JSON-RPC X ` 中不提倡使用 `批量调用`。

## 示例

Syntax:

```
--> data sent to Server
<-- data sent to Client
```

带索引数组参数的rpc调用:

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

带关联数组参数的rpc调用:

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

链式调用的rpc调用:

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

含生成实例的链式调用的rpc调用:

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

通知:

```python
--> {"jsonrpc": "X", "method": ["update"], "params": [1,2,3,4,5]}
--> {"jsonrpc": "X", "method": ["foobar"]}
```

不包含调用方法的rpc调用:

```python
--> {"jsonrpc": "X", "method": ["foobar"], "id": "1"}
<-- {"jsonrpc": "X", "error": {"code": -32601, "message": "Method not found"}, "id": "1"}
```

包含无效json的rpc调用:

```python
--> {"jsonrpc": "X", "method": "foobar, "params": "bar", "baz]
<-- {"jsonrpc": "X", "error": {"code": -32700, "message": "Parse error"}, "id": null}
```

包含无效请求对象的rpc调用:

```python
--> {"jsonrpc": "X", "method": 1, "params": ["bar"]}
<-- {"jsonrpc": "X", "error": {"code": -32600, "message": "Invalid Request"}, "id": null}
```

包含无效json的rpc批量调用:

```python
--> [
        {"jsonrpc": "X", "method": ["sum"], "params": [1,2,4], "id": "1"},
        {"jsonrpc": "X", "method"
    ]
<-- {"jsonrpc": "X", "error": {"code": -32700, "message": "Parse error"}, "id": null}
```

包含空数组的rpc调用:

```python
--> []
<-- {"jsonrpc": "X", "error": {"code": -32600, "message": "Invalid Request"}, "id": null}
```

非空且无效的rpc批量调用:

```python
--> [1]
<-- [
    {"jsonrpc": "X", "error": {"code": -32600, "message": "Invalid Request"}, "id": null}
    ]
```

无效的rpc批量调用:

```python
--> [1,2,3]
<-- [
    {"jsonrpc": "X", "error": {"code": -32600, "message": "Invalid Request"}, "id": null},
    {"jsonrpc": "X", "error": {"code": -32600, "message": "Invalid Request"}, "id": null},
    {"jsonrpc": "X", "error": {"code": -32600, "message": "Invalid Request"}, "id": null}
    ]
```

rpc批量调用:

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

所有都为通知的rpc批量调用:

```python
--> [
    {"jsonrpc": "X", "method": ["notify_sum"], "params": [1,2,4]},
    {"jsonrpc": "X", "method": ["notify_hello"], "params": [7]}
]

<-- //Nothing is returned for all notification batches
```

## 扩展

以rpc开头的方法名预留作为系统扩展，且必须不能用于其他地方。每个系统扩展都应该有相关规范文档，所有系统扩展都应是可选的。

Copyright (C) 2007-2010 by the JSON-RPC Working Group

This document and translations of it may be used to implement JSON-RPC, it may be copied and furnished to others, and derivative works that comment on or otherwise explain it or assist in its implementation may be prepared, copied, published and distributed, in whole or in part, without restriction of any kind, provided that the above copyright notice and this paragraph are included on all such copies and derivative works. However, this document itself may not bemodified in any way.

---

本页面的文字允许在知识共享 [署名-相同方式共享 3.0协议](http://zh.wikipedia.org/wiki/Wikipedia:CC-BY-SA-3.0%E5%8D%8F%E8%AE%AE%E6%96%87%E6%9C%AC)和[GNU自由文档许可证](http://zh.wikipedia.org/wiki/Wikipedia:GNU%E8%87%AA%E7%94%B1%E6%96%87%E6%A1%A3%E8%AE%B8%E5%8F%AF%E8%AF%81%E6%96%87%E6%9C%AC)下修改和再使用
