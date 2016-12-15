---
layout: default
title: Fastest JSON parser ever
---

jsoniter（json-iterator）是一款快且灵活的 JSON 解析器，同时提供 [Java](https://github.com/json-iterator/java) 和 [Go](https://github.com/json-iterator/go) 两个版本

# jsoniter 有何独特之处？

* Jsoniter 是最快的 JSON 解析器。它最多能比普通的解析器快10倍之多，即使在数据绑定的用法下也有同样的性能优势。无耻地献上自己的 [跑分](/benchmark.html)
* 非常易于使用的 api，允许你使用任何风格或者混搭的方式来解析 JSON。给你前所未有的灵活性。来看看这些 [API们](/api.html) 是不是真的有那么好用吧
* 独特的 iterator api 能够直接遍历 JSON，极致性能！0 内存分配！这样的 [iterator](/api.html#iterator-api) 你绝对没有用过

# 自卖自夸

忍不住要显摆一下。完整报告请看[性能评测](/benchmark.html)，对于[性能优化是怎么做的](/benchmark.html#optimization-used)有详尽的解释。

Java 版本数据绑定用法下的性能

![java1](http://jsoniter.com/benchmarks/java1.png)

Go 版本数据绑定用法下的性能

![go-medium](http://jsoniter.com/benchmarks/go-medium.png)

# Bind-API 最熟悉的老味道

没有特殊需求的时候，应该作为默认的最佳选择。对于这个 JSON 文档 `[0,1,2,3]`

使用 Java bind-api

```java
import com.jsoniter.Jsoniter;
Jsoniter iter = Jsoniter.parse("[0,1,2,3]");
int[] val = iter.read(int[].class);
System.out.println(val[3]);
```

使用 Go bind-api

```go
import "github.com/json-iterator/go"
iter := jsoniter.ParseString(`[0,1,2,3]`)
val := []int{}
iter.Read(&val)
fmt.Println(val[3])
```

# Iterator-API 用于快速抽取数据

不用把数据全部读出来，只是选择性抽取

使用 Java iterator-api

```java
import com.jsoniter.Jsoniter;
Jsoniter iter = Jsoniter.parse("[0, [1, 2], [3, 4], 5]");
int count = 0;
while(iter.readArray()) {
    iter.skip();
    count++;
}
System.out.println(count); // 4
```

使用 Go iterator-api

```go
import "github.com/json-iterator/go"
iter := ParseString(`[0, [1, 2], [3, 4], 5]`)
count := 0
for iter.ReadArray() {
    iter.skip()
    count++
}
fmt.Println(count) // 4
```
# Any-API 具有最好的灵活性

使用 Java any-api

```java
import com.jsoniter.Jsoniter;
Jsoniter iter = Jsoniter.parse("[{'field1':'11','field2':'12'},{'field1':'21','field2':'22'}]".replace('\'', '"'));
Any val = iter.readAny();
System.out.println(val.toInt(1, "field2")); // 22
```

注意你可以从嵌套的结构中直接取数据出来，并且转换成任意你想要的类型。

使用 Go any-api

```go
import "github.com/json-iterator/go"
iter := jsoniter.ParseString(`[{"field1":"11","field2":"12"},{"field1":"21","field2":"22"}]`)
val := iter.ReadAny()
fmt.Println(val.ToInt(1, "field2")) // 22
```

# 怎样获取

Java 版本

```
<dependency>
    <groupId>com.jsoniter</groupId>
    <artifactId>jsoniter</artifactId>
    <version>0.9.1</version>
</dependency>
```

Go 版本

```
go get github.com/json-iterator/go
```

# 欢迎你的贡献

小项目bug难免，欢迎提 issue 和 pull request
