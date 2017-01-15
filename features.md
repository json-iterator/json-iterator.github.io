---
layout: default
title: Jsoniter Features (Java Version)
---

# Core

| feature | java | go | note | 
| --- | --- | --- | --- |
| read float32 | Yes |  Yes | fast path: if the number is 0~9 only, read it into long, then divide by "power of 10", otherwise fallback to Float.valueOf. table lookup: given c, tell if it is 0~9, or dot, or end of number by a pre-defined table. |
| read float32 (streaming) | Yes | Yes | assume the number fit in current buffer, if it turns out the number end not found, fallback to slow path, read byte by byte then Float.valueOf. read from iterator head to tail, then load more, which skipped checking tail for each byte. |
| read float64 | Yes |  | same as above |
| read float64 (streaming) | Yes |  | same as above |
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
