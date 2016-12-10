---
layout: default
title: Json Iterator
---

* TOC
{:toc}

# Java Bind API

## 1 kb

| parser | score |
| ---    | ---   |
| jackson  | 850892.349 ± 39530.833  ops/s |
| gson     | 392361.207 ± 17619.850  ops/s |
| fastjson | 544556.920 ± 61141.858  ops/s |
| dsljson  | 1335352.551 ± 24010.110  ops/s |
| jsoniter | 4933967.110 ± 138318.632  ops/s |

## 10 kb


| parser | score |
| ---    | ---   |
| jackson  | 100824.573 ± 3038.689  ops/s |
| gson     | 49527.521 ± 2497.945  ops/s |
| fastjson | 72793.357 ± 3354.573  ops/s |
| dslsjon  | 164668.349 ±  7329.267  ops/s |
| jsoniter | 531711.831 ± 40921.227  ops/s |

## 100 kb

| parser | score |
| ---    | ---   |
| jackson  | 9683.500 ±  797.632  ops/s |
| gson     | 4959.971 ±  367.413  ops/s |
| fastjson | 6944.306 ±  456.864  ops/s |
| dslsjon  | 16793.305 ±  627.311  ops/s |
| jsoniter | 54352.743 ± 2239.098  ops/s |

# Go Bind API

* encoding/json: reflection
* easyjson: go generate
* jsonparser: hand-written data bind
* jsoniter (iterator-api): hand-written data bind
* jsoniter (bind-api): reflection

## Small Payload

| parser                  | ns/op      | bytes/op | allocs/op   |
| ---                     | ---        | ---      | ---         |
| encoding/json           | 3151 ns/op | 480 B/op |	6 allocs/op |
| easyjson                | 786 ns/op	 | 64 B/op	| 2 allocs/op |
| jsonparser              | 718 ns/op	 | 64 B/op  | 2 allocs/op |
| jsoniter (iterator-api) | 619 ns/op  | 64 B/op  | 2 allocs/op |
| jsoniter (bind-api)     | 844 ns/op  | 256 B/op | 4 allocs/op |

## Medium Payload

| parser                  | ns/op       | bytes/op | allocs/op    |
| ---                     | ---         | ---      | ---          |
| encoding/json           | 30531 ns/op	| 808 B/op | 18 allocs/op |
| easyjson                | 7731 ns/op  | 248 B/op | 8 allocs/op  |
| jsonparser              | 6326 ns/op  | 104 B/op | 4 allocs/op  |
| jsoniter (iterator-api) | 4966 ns/op	| 104 B/op | 4 allocs/op  |
| jsoniter (bind-api)     | 5640 ns/op  | 368 B/op | 14 allocs/op |

