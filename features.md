---
layout: default
title: Jsoniter Features (Java Version)
---

# Core

| feature | java | go | note | 
| --- | --- | --- | --- |
| read float32 | Yes |  Yes | fast path: if the number is 0~9 only, read it into long, then divide by "power of 10", otherwise fallback to Float.valueOf. table lookup: given c, tell if it is 0~9, or dot, or end of number by a pre-defined table. |
| read float32 (streaming) | Yes | Yes | assume the number fit in current buffer, if it turns out the number end not found, fallback to slow path, read byte by byte then Float.valueOf. read from iterator head to tail, then load more, which skipped checking tail for each byte. |
| read float64 | Yes | Yes | same as above |
| read float64 (streaming) | Yes | Yes | same as above |
| read int32 | Yes | Yes | value = value * 10 + ind, scan directly from bytes, no string allocation; pre-defined table to tell if it is a digit, and what is the value |
| read int32 (streaming) | Yes | Yes | use index to access buf, do not use readByte |
| read int64 | Yes | Yes | same as above |
| read int64 (streaming) | Yes | Yes | same as above |
| read string | Yes | Yes | fast path for ascii and no escape string, fallback to slow path resuming from the break point. java string is utf16 based, so \u is quicker. golang string is utf8 based, so \u convert utf16 to utf8, but other utf8 bytes are quicker. java use IndexOutOfBound exception to handle looking ahead problem, avoid read byte by byte |
| read string (streaming) | Yes | Yes | read byte by byte |
| read string as slice | Yes | Yes | for certain string input (object field, base64), we know it will not contain any escape or utf8, so it is a direct copy from byte to char array. The original byte array is not copied but reused as slice |
| read string as slice (streaming) | Yes | Yes | If the buffer is large enough, byte array is reused as slice. Otherwise copy will happen. TODO: avoid small memory allocation |
| read true/false/null | Yes | Yes | skip fixed bytes
| read true/false/null (streaming) | Yes | Yes | load buffer once should be bigger enough to skip all the bytes |
| read object | Yes | Yes | |
| read object callback | Yes | Yes | callback should be faster, as less branching in the code |
| read array | Yes | Yes | |
| read array callback | Yes | Yes | callback should be faster, as less branching in the code |
| skip whitespace | Yes | Yes | |
| skip whitespace (streaming) | Yes | Yes | |
| skip string | Yes | Yes | find the end of string, and move pointer. if last byte is \, need to check if it is \\ or \\\ |
| skip string (streaming) | Yes | Yes | the \ on the boundary is a problem |
| skip number | Yes | Yes | skip until break |
| skip number (streaming) | Yes | Yes | |
| skip array/object | Yes | Yes | |
| skip array/object (streaming) | Yes | Yes | |
| skip true/false/null | Yes | Yes | skip fixed bytes |
| skip true/false/null (streaming) | Yes | Yes | might across the boundary at most once |
| whatIsNext | Yes | Yes | lookup with a table |
| read object/interface{} | Yes | Yes | |
| write float32 | Yes | Yes | only keep 6 digits |
| write float64 | Yes | Yes | only keep 6 digits |
| write int32 | Yes | Yes | encode "000" three digits as ascii in one int value from 0 ~ 999, then process by /1000 each time |
| write int64 | Yes | Yes | same as above |
| write ascii string | Yes |  |
| write string | Yes |  |
| write true/false/null | Yes |  |
| write array/object | Yes |  |
| write whitespace | Yes |  |

# Any

| feature | java | go | note | 
| --- | --- | --- | --- |
| any lazy array | Yes |  |
| any lazy object | Yes |  |
| any others | Yes |  |

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

# TO DO

* avoid small memory allocation, maintain new buffer internally (for streaming read bytes, and any)
* per byte field matching in one scan, fast path if the buffer is large enough (or if streaming support is on)
* write to byte array without checking length, pre-allocate large enough buffer

