---
layout: default
title: Jsoniter Features (Java Version)
---

# Core

| feature | java | go |
| --- | --- | --- |
| stream | Yes | No |
| read float32 | Yes | Yes |
| read float32 (streaming) | Yes | Yes |
| read float64 | Yes | Yes |
| read float64 (streaming) | Yes | Yes |
| read int32 | Yes | Yes |
| read int32 (streaming) | Yes | Yes |
| read int64 | Yes | Yes |
| read int64 (streaming) | Yes | Yes |
| read string | Yes | Yes |
| read string (streaming) | Yes | Yes |
| read string as slice | Yes | Yes |
| read string as slice (streaming) | Yes | Yes |
| read true/false/null | Yes | Yes |
| read true/false/null (streaming) | Yes | Yes |
| read array/object | Yes | Yes |
| skip whitespace | Yes | Yes |
| skip string | Yes | Yes |
| skip string (streaming) | Yes | Yes |
| skip number | Yes | Yes |
| skip number (streaming) | Yes | Yes |
| skip array/object | Yes | Yes |
| skip array/object (streaming) | Yes | Yes |
| skip true/false/null | Yes | Yes |
| skip true/false/null (streaming) | Yes | Yes |
| whatIsNext | Yes | Yes |
| read object/interface{} | Yes | Yes |
| any lazy array | Yes | Yes |
| any lazy object | Yes | Yes |
| any others | Yes | Yes |
| small allocation optimization (streaming) | No | No |
| write float32 | Yes | Yes |
| write float64 | Yes | Yes |
| write int32 | Yes | Yes |
| write int64 | Yes | Yes |
| write ascii string | Yes | Yes |
| write string | Yes | Yes |
| write true/false/null | Yes | Yes |
| write array/object | Yes | Yes |
| write whitespace | Yes | Yes |

# Object Decoding

| feature | java reflection | java codegen | go reflection |
| --- | --- | --- | --- |
| field binding | Yes | Yes | Yes |
| setter binding | Yes | Yes | N/A |
| ctor/factory/di support | Yes | Yes | N/A |
| wrapper | Yes | Yes | N/A |
| validation | Yes | Yes | No |
| array/collection support | Yes | Yes | No |
| map support | Yes | Yes | No |
| enum support | Yes | Yes | No |
| int support | Yes | Yes | No |
| float support | Yes | Yes | No |
| string support | Yes | Yes | No |
| boolean support | Yes | Yes | No |

Go type embeding support?

# Object Encoding 

| feature | java reflection | java codegen | go reflection |
| --- | --- | --- | --- |
| field binding | Yes | Yes | No |
| getter binding | Yes | Yes | N/A |
| unwrapper | Yes | Yes | N/A |
| array/collection/list support | Yes | Yes | No |
| map support | Yes | Yes | No |
| enum support | Yes | Yes | No |
| int support | Yes | Yes | No |
| float support | Yes | Yes | No |
| string support | Yes | Yes | No |
| boolean support | Yes | Yes | No |
