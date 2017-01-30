---
layout: default
title: Fastest JSON parser ever
---

jsoniter (json-iterator) is fast and flexible JSON parser available in [Java](https://github.com/json-iterator/java) and [Go](https://github.com/json-iterator/go). Good deal of the code is ported from [dsljson](https://github.com/ngs-doo/dsl-json), and [jsonparser](https://github.com/buger/jsonparser).

# Faster, Much Faster!

Mainstream JSON parsers are slow. Jsoniter Java version could be 5x times faster than jackson/gson/fastjson. If you are doing a lot of log processing or number crunching, but stuck with JSON, you definitely need to consider [dsl-json](https://github.com/ngs-doo/dsl-json) or Jsoniter to save the encoding/decoding cost.

![java1](http://jsoniter.com/benchmarks/java1.png)

Jsoniter Golang version could be more than 6x times faster than standard lib (encoding/json). And the number is acheived with runtime reflection instead of `go generate`.

![go-medium](http://jsoniter.com/benchmarks/go-medium.png)

For more complete report you can checkout the full [benchmark](/benchmark.html) with [in-depth optimization](/benchmark.html#optimization-used) to back the numbers up

# Super flexible API

* any-api: use Java like PHP, high performance by lazy parsing
* iterator-api: read through the JSON just like iterating over a collection
* bind-api: binding any kind of data structure. It can even bind to existing object
 
Here is a simple demo. Each line is a object, first element being the order id, second element being the order details

```json
[1024, {"product_id": 100, "start": "beijing"}]
["1025", {"product_id": 101, "start": "shanghai"}]
// many many more lines
```

There are three things to notice

* There are many lines, read them all in once will have memory issue
* Some order id is int, some order id is string. This is very common when working with PHP.
* The order details has many fields, need object binding

Amazingly, in 6 lines, we have all problems solved:

```java
JsonIterator iter = JsonIterator.parse(input);
OrderDetails orderDetails = new OrderDetails();
while(iter.whatIsNext() != ValueType.INVALID) {
    Any order = iter.readAny();
    int orderId = order.toInt(0);
    String start = order.get(1).bindTo(orderDetails).start;
}
```

* JsonIterator.parse take InputStream as input, parse everything in a streaming way
* readAny returns an instance of Any. The parsing is lazily done when actually getting the field, simple and performant.
* bindTo(orderDetails), data binding can reuse existing object

Good old one line api is also available. To serialize

```java
JsonStream.serialize(new int[]{1,2,3})
```

To deserialize

```java
JsonIterator.deserialize("[1,2,3]", int[].class)
```

[More on awsome apis](/java-features.html)

# How to get

For Java version

```
<dependency>
    <groupId>com.jsoniter</groupId>
    <artifactId>jsoniter</artifactId>
    <version>0.9.7</version>
</dependency>
```

For Go version

```
go get github.com/json-iterator/go
```

# Contribution Welcomed !

Report issue or pull request, or email taowen@gmail.com, or [![Gitter chat](https://badges.gitter.im/gitterHQ/gitter.png)](https://gitter.im/json-iterator/Lobby)
