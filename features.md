---
layout: default
title: Jsoniter Features (Java Version)
---

# Core

| feature | java | go | note | 
| --- | --- | --- | --- |
| read float32 | Yes |  |
| read float32 (streaming) | Yes |  |
| read float64 | Yes |  |
| read float64 (streaming) | Yes |  |
| read int32 | Yes |  |
| read int32 (streaming) | Yes |  |
| read int64 | Yes |  |
| read int64 (streaming) | Yes |  |
| read string | Yes |  |
| read string (streaming) | Yes |  |
| read string as slice | Yes |  |
| read string as slice (streaming) | Yes |  |
| read true/false/null | Yes |  |
| read true/false/null (streaming) | Yes |  |
| read array/object | Yes |  |
| skip whitespace | Yes |  |
| skip string | Yes |  |
| skip string (streaming) | Yes |  |
| skip number | Yes |  |
| skip number (streaming) | Yes |  |
| skip array/object | Yes |  |
| skip array/object (streaming) | Yes |  |
| skip true/false/null | Yes |  |
| skip true/false/null (streaming) | Yes |  |
| whatIsNext | Yes |  |
| read object/interface{} | Yes |  |
| any lazy array | Yes |  |
| any lazy object | Yes |  |
| any others | Yes |  |
| small allocation optimization (streaming) | No |  |
| write float32 | Yes |  |
| write float64 | Yes |  |
| write int32 | Yes |  |
| write int64 | Yes |  |
| write ascii string | Yes |  |
| write string | Yes |  |
| write true/false/null | Yes |  |
| write array/object | Yes |  |
| write whitespace | Yes |  |

# Object Decoding

| feature | java reflection | java codegen | go reflection |
| --- | --- | --- | --- |
| field binding | Yes | Yes |  |
| setter binding | Yes | Yes |  |
| ctor/factory/di support | Yes | Yes | N/A |
| wrapper | Yes | Yes | N/A |
| validation | Yes | Yes |  |
| array/collection support | Yes | Yes |  |
| map support | Yes | Yes |  |
| enum support | Yes | Yes |  |
| int support | Yes | Yes |  |
| float support | Yes | Yes | |
| string support | Yes | Yes |  |
| boolean support | Yes | Yes |  |

Go type embeding support?

# Object Encoding 

| feature | java reflection | java codegen | go reflection |
| --- | --- | --- | --- |
| field binding | Yes | Yes |  |
| getter binding | Yes | Yes | N/A |
| unwrapper | Yes | Yes | N/A |
| array/collection/list support | Yes | Yes |  |
| map support | Yes | Yes |  |
| enum support | Yes | Yes |  |
| int support | Yes | Yes |  |
| float support | Yes | Yes |  |
| string support | Yes | Yes |  |
| boolean support | Yes | Yes |  |
| indention support | Yes | No |  |
