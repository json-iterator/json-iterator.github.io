---
layout: default
title: Fastest JSON parser ever
---

# jsoniter 有何独特之处？

* Jsoniter 是最快的 JSON 解析器。它能比普通的解析器最高快到10倍之多，即使在数据绑定的用法下也有同样的性能差距。（无耻地献上自己的 [跑分](/benchmark.html)）
* 使用起来容易始终是我们的第一优先级，你可以从 bind-api, any-api 或者 iterator-api 中选择合适的，或者全都用上（来看看这些 [API们](/api.html) 是不是真的有那么好用吧）
* 独特的 iterator api 可以直接遍历 JSON，可以做到 0 内存分配！（这里讲解了 [iterator](/api.html#iterator-api) 是如何工作的）

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
