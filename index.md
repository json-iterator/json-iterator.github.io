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
import com.jsoniter.Jsoniter
Jsoniter iter = Jsoniter.parse("[0,1,2,3]");
int[] val = iter.read(int[].class);
System.out.println(val[3]);
```

Parse with Golang bind-api

```go
import "github.com/json-iterator/go"
iter := jsoniter.ParseString(`[0,1,2,3]`)
val := []int{}
iter.Read(&val)
fmt.Println(val[3])
```

# Contribution Welcomed !

Report issue or pull request, or email taowen@gmail.com
