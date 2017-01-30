---
layout: default
title: Fastest JSON parser ever
---

jsoniter (json-iterator) is fast and flexible JSON parser available in [Java](https://github.com/json-iterator/java) and [Go](https://github.com/json-iterator/go). Good deal of the code is ported from [dsljson](https://github.com/ngs-doo/dsl-json), and [jsonparser](https://github.com/buger/jsonparser).

# Faster, Much Faster!

Mainstream JSON parsers are very slow. Jsoniter Java version could be 4x times faster than jackson/gson/fastjson. If you are doing a lot of log processing or number crunching, but stuck with JSON, you definitely need to consider [dsl-json](https://github.com/ngs-doo/dsl-json) or Jsoniter to save the encoding/decoding cost. According to dsl-json, the JSON encoding/decoding speed is not a problem, [at least faster than thrift/avro](https://www.codeproject.com/Articles/1165627/Jsoniter-JSON-is-faster-than-thrift-avro)

![java1](http://jsoniter.com/benchmarks/java1.png)

Jsoniter Golang version could be more than 6x times faster than standard lib (encoding/json). And the number is acheived with runtime reflection instead of `go generate`.

![go-medium](http://jsoniter.com/benchmarks/go-medium.png)

For more complete report you can checkout the full [benchmark](/benchmark.html) with [in-depth optimization](/benchmark.html#optimization-used) to back the numbers up

# Cut the Crap!

Jsoniter gets things done, as fast as possible. Most common use case is just one line:

```java
JsonStream.serialize(new int[]{1,2,3});
```

```java
JsonIterator.deserialize("[1,2,3]", int[].class);
```

According to your past experience, you must know the following code will be very slow and cumbersome:

```java
Map<String, Object> obj = deserialize(input);
Object firstItem = ((List<Object>)obj.get("items")).get(0);
```

To get the best performance and concise compact code, you'd better define a class as the data schema:

```java
public class Order {
public List<OrderEntry> items;
}
Order order = deserialize(input, Order.class);
OrderEntry firstItem = obj.items.get(0);
```

This is a good practice when you are doing very formal business logic. But what if you just want to get a string from a deeply nested structure, will you have the patience to define the data schema of each level? Cut the crap, please! Most data extraction can be done in one line:

```java
Jsoniter.deserialize(input).get("items", 0); // the first item
```

Jsoniter will not only be the fastest parser in runtime, but also be the fastest parser to help you getting your job done.

# Documentation

Jsoniter has more than enough feature, with an example driven style document. You can see a lot of code snippet to demonstrate various common task:

* [How to serialzie/deserialize in one line](http://jsoniter.com/java-features.html#very-simple-api)
* [How to use in Android platform with static code generation](http://jsoniter.com/java-features.html#performance-is-optional)
* [How to check if property is present in JSON](http://jsoniter.com/java-features.html#validation)
* [How to customize encoding/decoding](http://jsoniter.com/java-features.html#service-provider-interface-spi)
* [Many more...](http://jsoniter.com/java-features.html)

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
