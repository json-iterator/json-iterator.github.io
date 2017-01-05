---
layout: default
title: Fastest JSON parser ever
---

jsoniter (json-iterator) is fast and flexible JSON parser available in [Java](https://github.com/json-iterator/java) and [Go](https://github.com/json-iterator/go)

# Why jsoniter?

* Jsoniter is the fastest JSON decoder & encoder. It could be up to 10x faster than normal parser, data binding included. Shameless self [benchmark](/benchmark.html)
* Extremely flexible api. You can mix and match three different styles: bind-api, any-api or iterator-api. Checkout your [api choices](/java-features.html)

# Show off

Here is a quick show off, for more complete report you can checkout the full [benchmark](/benchmark.html) with [in-depth optimization](/benchmark.html#optimization-used) to back the numbers up

This is java version, doing data binding

![java1](http://jsoniter.com/benchmarks/java1.png)

This is go version, doing data binding

![go-medium](http://jsoniter.com/benchmarks/go-medium.png)

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

[More on awsome apis](/java-features.cn.html)

# How to get

For java version

```
<dependency>
    <groupId>com.jsoniter</groupId>
    <artifactId>jsoniter</artifactId>
    <version>0.9.5</version>
</dependency>
```

For Go version

```
go get github.com/json-iterator/go
```

# Contribution Welcomed !

Report issue or pull request, or email taowen@gmail.com, or [![Gitter chat](https://badges.gitter.im/gitterHQ/gitter.png)](https://gitter.im/json-iterator/Lobby)
