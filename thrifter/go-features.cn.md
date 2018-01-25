---
layout: default
title: Thrifter Features (Go Version)
---

* TOC
{:toc}

# 超简单的 api

```go
// 序列化成 thrift 协议
thriftEncodedBytes, err := Thrifter.Marshal([]int{1, 2, 3})
// 反序列化回 go 对象
var val []int
err = Thrifter.Unmarshal(thriftEncodedBytes, &val)
```
