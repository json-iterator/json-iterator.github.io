---
layout: default
title: Fastest JSON parser ever
---

# jsoniter 有何独特之处？

* Jsoniter 是最快的 JSON 解析器。它最多能比普通的解析器快10倍之多，即使在数据绑定的用法下也有同样的性能优势。（无耻地献上自己的 [跑分](/benchmark.html)）
* 非常易于使用的 api，允许你使用任何风格或者混搭的方式来解析 JSON。给你前所未有的灵活性。（来看看这些 [API们](/api.html) 是不是真的有那么好用吧）
* 独特的 iterator api 能够直接遍历 JSON，极致性能！0 内存分配！（这里讲解了 [iterator](/api.html#iterator-api) 是如何工作的）

同时提供 Java 和 Go 两个版本

# 1 分钟教程

对于这个 JSON 文档 `[0,1,2,3]`

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

使用 Java any-api

```java
import com.jsoniter.Jsoniter;
Jsoniter iter = Jsoniter.parse("[0,1,2,3]");
Any val = iter.readAny();
System.out.println(any.get(3));
```

使用 Go any-api

```go
import "github.com/json-iterator/go"
iter := jsoniter.ParseString(`[0,1,2,3]`)
val := iter.ReadAny()
fmt.Println(val.Get(3))
```

使用 Java iterator-api

```java
import com.jsoniter.Jsoniter;
Jsoniter iter = Jsoniter.parse("[0,1,2,3]");
int total = 0;
while(iter.readArray()) {
    total += iter.readInt();
}
System.out.println(total);
```

使用 Go iterator-api

```go
import "github.com/json-iterator/go"
iter := ParseString(`[0,1,2,3]`)
total := 0
for iter.ReadArray() {
    total += iter.ReadInt()
}
fmt.Println(total)
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
