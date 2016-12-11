---
layout: default
title: Fastest JSON parser ever
---

# Why json iterator (jsoniter)?

* Jsoniter is the fastest JSON parser, it could be up to 10x faster than normal parser, data binding included (shameless self [benchmark](/benchmark.html))
* Having a developer friendly api is our #1 prioprity, you can choose from bind-api, any-api or iterator-api (checkout your [api choices](/api.html))
* Unique iterator api can iterate through JSON directly, zero memory allocation! (see how [iterator](/api.html) works)

Available today, as both Java and Go version.

# 1 Minute Tutorial

Given this JSON document `[0,1,2,3]`

Parse with Java bind-api

```java
import com.jsoniter.Jsoniter;
Jsoniter iter = Jsoniter.parse("[0,1,2,3]");
int[] val = iter.read(int[].class);
System.out.println(val[3]);
```

Parse with Go bind-api

```go
import "github.com/json-iterator/go"
iter := jsoniter.ParseString(`[0,1,2,3]`)
val := []int{}
iter.Read(&val)
fmt.Println(val[3])
```

Parse with Java any-api

```java
import com.jsoniter.Jsoniter;
Jsoniter iter = Jsoniter.parse("[0,1,2,3]");
Any val = iter.readAny();
System.out.println(any.get(3));
```

Parse with Go any-api

```go
import "github.com/json-iterator/go"
iter := jsoniter.ParseString(`[0,1,2,3]`)
val := iter.ReadAny()
fmt.Println(val.Get(3))
```

Parse with Java iterator-api

```java
import com.jsoniter.Jsoniter;
Jsoniter iter = Jsoniter.parse("[0,1,2,3]");
int total = 0;
while(iter.readArray()) {
    total += iter.readInt();
}
System.out.println(total);
```

Parse with Go iterator-api

```go
import "github.com/json-iterator/go"
iter := ParseString(`[0,1,2,3]`)
total := 0
for iter.ReadArray() {
    total += iter.ReadInt()
}
fmt.Println(total)
```

# How to get

For java version

```
<dependency>
    <groupId>com.jsoniter</groupId>
    <artifactId>jsoniter</artifactId>
    <version>0.9.1</version>
</dependency>
```

For Go version

```
go get github.com/json-iterator/go
```

# Contribution Welcomed !

Report issue or pull request, or email taowen@gmail.com