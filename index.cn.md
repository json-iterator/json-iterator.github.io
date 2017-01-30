---
layout: default
title: Fastest JSON parser ever
---

jsoniter（json-iterator）是一款快且灵活的 JSON 解析器，同时提供 [Java](https://github.com/json-iterator/java) 和 [Go](https://github.com/json-iterator/go) 两个版本。从 [dsljson](https://github.com/ngs-doo/dsl-json) 和 [jsonparser](https://github.com/buger/jsonparser) 借鉴了大量代码。

# 就是快！

主流的 JSON 解析器是非常慢的。Jsoniter Java 版本可以比常用的 jackson/gson/fastjson **快 4 倍**。如果你需要处理大量的 JSON 格式的日志，你应该考虑一下用 [dsl-json](https://github.com/ngs-doo/dsl-json) 或者 Jsoniter 来节约可观的成本。根据 dsl-json 的性能评测，JSON 格式序列化和反序列化的速度其实一点都不慢，[甚至比 thrift/avro 还要快](https://www.codeproject.com/Articles/1165627/Jsoniter-JSON-is-faster-than-thrift-avro)。

![java1](http://jsoniter.com/benchmarks/java1.png)

Jsoniter 的 Golang 版本可以比标准库（encoding/json）**快 6 倍**之多。而且这个性能是在不使用代码生成的前提下获得的。

![go-medium](http://jsoniter.com/benchmarks/go-medium.png)

完整报告请看[性能评测](/benchmark.html)，对于[性能优化是怎么做的](/benchmark.html#optimization-used)有详尽的解释。

# 独特体验

Jsoniter 的目标就是帮你把事搞定，越快越好。最常见的用法只需要一行：

```java
JsonStream.serialize(new int[]{1,2,3}); // from object to JSON
```

```java
JsonIterator.deserialize("[1,2,3]", int[].class); // from JSON to object, with class specified
```
如果这就是 Jsoniter 的一切本领，那么它不过是个平庸之辈而已。然而 Jsoniter 源于作者使用现有解析器时的不满与愤怒，它绝不会别人的老路的。想要体会到 Jsoniter 的独特体验能带来什么，我们来比较一下现有常规的 JSON API 的使用体验。

根据过去的老经验，你一定知道下面这种用法是效率很低而且笨拙的，但是有些时候又不得不这么用：

```java
Map<String, Object> obj = deserialize(input);
Object firstItem = ((List<Object>)obj.get("items")).get(0);
```

想要最佳的性能以及代码工整，你最好定义一个类来指定数据的格式：

```java
public class Order {
  public List<OrderEntry> items;
}
Order order = deserialize(input, Order.class);
OrderEntry firstItem = obj.items.get(0);
```
在写正式的业务逻辑的代码时，这当然是很好的实践。但是如果你只是想从一个JSON嵌套结构里取一个内部的字符串的值的时候，必须提前定义每层数据结构未免有点太费周章了。能一行搞定的，就别费那么些话了：

```java
Jsoniter.deserialize(input).get("items", 0); // the first item
```

deserialize 的返回值类型是“Any”，它有点类似于 `Map<String, Object>`。两者都是通用的数据容器，但是和  `Map<String, Object>` 不同，Any 有通过  api 使得数据获取上更方便：

```java
Any any = Jsoniter.deserialize(input); // deserialize 返回 "Any"，实际的解析是延迟在读取时才做的
any.get("items", '*', "name", 0); // 抽取所有 items 的第一个 name
any.get("size").toLong(); // 不管是 "100" 还是 100，都给转成 long 类型，就像弱类型一样
any.bindTo(Order.class); // 把 JSON 绑定到对象
for (Any element : any) {} // 遍历集合，Any 实现了 iterable 接口
```

更好的消息是，这种 schema-less 的体验在延迟解析技术的帮助下，做到了性能上的无损。所有没有别读取的字段，仍然会以 JSON 的原始格式保留。使用 `Any` 的性能要比使用 `Map<String, Object>` 好得多。现在，在 Java 语言中，你也体会到 Javascript 或者 PHP 解析 JSON 时那种丝滑般体验。[JSON 与 any，乐趣多多](http://jsoniter.com/java-features.cn.html#section-19).

Jsoniter 不仅仅在运行时要做最快的解析器，也同时非常努力地变成代码写起来最方便的解析器。

# 文档

Jsoniter 功能多多，文档以例子为主。有很多代码示例来演示这些常用任务如何实现：

* [如何在 Android 平台上使用](http://jsoniter.com/java-features.cn.html#section-3)
* [如何检查 JSON 中是否包含指定属性](http://jsoniter.com/java-features.cn.html#section-10)
* [如何自定义序列化和反序列化的方法](http://jsoniter.com/java-features.cn.html#service-provider-interface-spi)
* [还有许多……](http://jsoniter.com/java-features.cn.html)

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
