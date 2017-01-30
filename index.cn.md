---
layout: default
title: Fastest JSON parser ever
---

jsoniter（json-iterator）是一款快且灵活的 JSON 解析器，同时提供 [Java](https://github.com/json-iterator/java) 和 [Go](https://github.com/json-iterator/go) 两个版本。从 [dsljson](https://github.com/ngs-doo/dsl-json) 和 [jsonparser](https://github.com/buger/jsonparser) 借鉴了大量代码。

# 就是快

主流的 JSON 解析器是非常慢的。Jsoniter Java 版本可以比常用的 jackson/gson/fastjson 快 4 倍。如果你需要处理大量的 JSON 格式的日志，你应该考虑一下用 [dsl-json](https://github.com/ngs-doo/dsl-json) 或者 Jsoniter 来节约可观的成本。根据 dsl-json 的性能评测，JSON 格式序列化和反序列化的速度其实一点都不慢，[甚至比 thrift/avro 还要快](https://www.codeproject.com/Articles/1165627/Jsoniter-JSON-is-faster-than-thrift-avro)。

![java1](http://jsoniter.com/benchmarks/java1.png)

Go 版本数据绑定用法下的性能

![go-medium](http://jsoniter.com/benchmarks/go-medium.png)

完整报告请看[性能评测](/benchmark.html)，对于[性能优化是怎么做的](/benchmark.html#optimization-used)有详尽的解释。

# 超级灵活的 API

* any-api：让你把 Java 用出 PHP 的感觉来，通过只解析用到字段来实现高性能
* iterator-api：读 JSON 就像在遍历一个集合一般简单
* bind-api：各种对象都可以绑定，还可以绑定到已有对象上
 
这三个 api 可以用一个很简单的例子来展示。这是一个记录了订单流水的 JSON 文件，每一行是一个订单。第一个元素是订单id，后面的是订单的详情。


```json
[1024, {"product_id": 100, "start": "beijing"}]
["1025", {"product_id": 101, "start": "shanghai"}]
// 此处省略几千行
```

这个文档有三处难点

* 行数非常多，如果一次性读到内存里可能会爆
* 其次同一个字段既可能是整数，也可能是字符串。很多 PHP 产生的 JSON 都有这个问题
* 详情部分可能字段比较多，需要绑定到对象上来处理

解析这个文档只需要6行

```java
JsonIterator iter = JsonIterator.parse(input);
OrderDetails orderDetails = new OrderDetails();
while(iter.whatIsNext() != ValueType.INVALID) {
    Any order = iter.readAny();
    int orderId = order.toInt(0);
    String start = order.get(1).bindTo(orderDetails).start;
}
```

* JsonIterator.parse 支持 InputStream 作为输入，完全流式解析
* readAny 解析为 Any 对象。实际的解析在 get 具体的字段的时候延迟触发。既方便，又高性能。
* bindTo(orderDetails) 数据绑定支持绑定到已有的对象上

当然最常用的还是这两个静态方法，序列化

```java
JsonStream.serialize(new int[]{1,2,3})
```

反序列化

```java
JsonIterator.deserialize("[1,2,3]", int[].class)
```

[更多 API 的用法参见手册](/java-features.cn.html)

# 怎样获取

Java 版本

```
<dependency>
    <groupId>com.jsoniter</groupId>
    <artifactId>jsoniter</artifactId>
    <version>0.9.7</version>
</dependency>
```

Go 版本

```
go get github.com/json-iterator/go
```

# 欢迎你的贡献

小项目bug难免，欢迎提 issue 和 pull request
