---
layout: default
title: Fastest JSON parser ever
---

jsoniter (json-iterator) is fast and flexible JSON parser available in [Java](https://github.com/json-iterator/java) and [Go](https://github.com/json-iterator/go). Good deal of the code is ported from [dsljson](https://github.com/ngs-doo/dsl-json), and [jsonparser](https://github.com/buger/jsonparser).

# Faster, Much Faster!

Traditional JSON parsers are slow. Jsoniter Java version could be **3x** times faster than jackson/gson/fastjson. If you are doing a lot of log processing or number crunching, but stuck with JSON, you definitely need to consider [dsl-json](https://github.com/ngs-doo/dsl-json) or Jsoniter to save the encoding/decoding cost. [Is protobuf 5x faster than JSON?](https://dzone.com/articles/is-protobuf-5x-faster-than-json), [part II](https://dzone.com/articles/is-protobuf-5x-faster-than-json-part-ii)

![protobuf-vs-jsoniter](http://jsoniter.com/benchmarks/protobuf-vs-jsoniter.png)

Jsoniter Golang version could be more than **6x** times faster than standard lib (encoding/json). And the number is acheived with runtime reflection instead of `go generate`.

![go-medium](http://jsoniter.com/benchmarks/go-medium.png)

For more complete report you can checkout the full [benchmark](/benchmark.html) with [in-depth optimization](/benchmark.html#optimization-used) to back the numbers up

# Parse, Just ONE Line!

Jsoniter gets things done, as fast as possible. Most common use case is just one line:

```java
JsonStream.serialize(new int[]{1,2,3}); // from object to JSON
```

```java
JsonIterator.deserialize("[1,2,3]", int[].class); // from JSON to object, with class specified
```

This is what a mediocre parser can do. Jsoniter is born from real-world anger to solve the impedance mismatch between JSON the Java language. To understand what kind of unique experience Jsoniter can provide, let's compare with existing parser api.

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

The return type of deserialize is "Any", very similiar to the idea of using `Map<String, Object>`, to represent Any data without defining the class. Unlike `Map<String, Object>`, it has a fluent api:

```java
Any any = Jsoniter.deserialize(input); // deserialize returns "Any", actual parsing is done lazily
any.get("items", '*', "name", 0); // extract out the first name from all items
any.get("size").toLong(); // no matter it is "100" or 100, return it as long, making Java weakly typed
any.bindTo(Order.class); // binding the JSON into object
for (Any element : any) {} // iterate the collection, Any implements iterable
```

Best of all, the schema-less experience is backed by lazy parsing technology. Anything not read, will be kept in JSON form as it is. The performance of `Any` will be much faster than using `Map<String, Object>`. Now, in Java, you can parse the JSON like Javascript or PHP. [JSON is fun with any](http://jsoniter.com/java-features.html#lazy-is-an-option).

Jsoniter will not only be the fastest parser in runtime, but also trying very hard to be the fastest parser to help you getting your job done.

# Documentation

Jsoniter has more than enough feature, with an example driven style document. You can see a lot of code snippet to demonstrate various common task:

* [How to use in Android platform with static code generation](http://jsoniter.com/java-features.html#performance-is-optional)
* [How to check if property is present in JSON](http://jsoniter.com/java-features.html#validation)
* [How to customize encoding/decoding](http://jsoniter.com/java-features.html#service-provider-interface-spi)
* [Many more...](http://jsoniter.com/java-features.html)

Tutorial on sitepoint: [https://www.sitepoint.com/php-style-json-parsing-in-java-with-jsoniter/](https://www.sitepoint.com/php-style-json-parsing-in-java-with-jsoniter/)

Introduction on dzone: [https://dzone.com/articles/is-protobuf-5x-faster-than-json](https://dzone.com/articles/is-protobuf-5x-faster-than-json)

# How to get

For Java version

```
<dependency>
    <groupId>com.jsoniter</groupId>
    <artifactId>jsoniter</artifactId>
    <version>0.9.13</version>
</dependency>
```

For Go version

```
go get github.com/json-iterator/go
```

# Contribution Welcomed !

Report issue or pull request, or email taowen@gmail.com, or [![Gitter chat](https://badges.gitter.im/gitterHQ/gitter.png)](https://gitter.im/json-iterator/Lobby)
