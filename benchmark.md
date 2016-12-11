---
layout: default
title: Json Iterator Benchmark
---

* TOC
{:toc}

Machine Used:

* CPU: i7-4790 @ 3.6G
* Memory: 16G

# Java Bind API

* JMH 1.17.1 (released 11 days ago)
* VM version: JDK 1.8.0_112, VM 25.112-b15
* VM invoker: /opt/jdk1.8.0_112/jre/bin/java
* VM options: -XX:+AggressiveOpts -Xms2G -Xmx2G
* Warmup: 3 iterations, 1 s each
* Measurement: 5 iterations, 2 s each
* Timeout: 10 min per iteration
* Threads: 16 threads, will synchronize iterations
* Benchmark mode: Throughput, ops/time

## 1 kb

[bind json to object](https://github.com/json-iterator/java-json-benchmark/blob/master/src/main/java/com/github/fabienrenaud/jjb/databind/Deserialization.java)

| parser | score |
| ---    | ---   |
| [jackson]  | 850892.349 ± 39530.833  ops/s |
| [gson]     | 392361.207 ± 17619.850  ops/s |
| [fastjson] | 544556.920 ± 61141.858  ops/s |
| [dsljson]  | 1335352.551 ± 24010.110  ops/s |
| jsoniter (bind-api) | 4933967.110 ± 138318.632  ops/s |

![java1](http://jsoniter.com/benchmarks/java1.png)

## 10 kb

[bind json to object](https://github.com/json-iterator/java-json-benchmark/blob/master/src/main/java/com/github/fabienrenaud/jjb/databind/Deserialization.java)

| parser | score |
| ---    | ---   |
| [jackson]  | 100824.573 ± 3038.689  ops/s |
| [gson]     | 49527.521 ± 2497.945  ops/s |
| [fastjson] | 72793.357 ± 3354.573  ops/s |
| [dsljson]  | 164668.349 ±  7329.267  ops/s |
| jsoniter (bind-api) | 531711.831 ± 40921.227  ops/s |

![java10](http://jsoniter.com/benchmarks/java10.png)

## 100 kb

[bind json to object](https://github.com/json-iterator/java-json-benchmark/blob/master/src/main/java/com/github/fabienrenaud/jjb/databind/Deserialization.java)

| parser | score |
| ---    | ---   |
| [jackson]  | 9683.500 ±  797.632  ops/s |
| [gson]     | 4959.971 ±  367.413  ops/s |
| [fastjson] | 6944.306 ±  456.864  ops/s |
| [dsljson]  | 16793.305 ±  627.311  ops/s |
| jsoniter (bind-api) | 54352.743 ± 2239.098  ops/s |

![java100](http://jsoniter.com/benchmarks/java100.png)

# Java Iterator API

## 1000 kb

[count number of elements from InputStream without binding](https://github.com/json-iterator/java-json-benchmark/blob/master/src/main/java/com/github/fabienrenaud/jjb/stream/UsersStreamDeserializer.java#L352)

| parser    | score |
| ---       | ---   |
| [javaxjson] | 1192.041 ± 71.521  ops/s  |
| [jackson]   | 1919.180 ± 122.895  ops/s |
| jsoniter (iterator-api) | 3165.283 ± 106.326  ops/s |

![java1000](http://jsoniter.com/benchmarks/java1000.png)

## 10000 kb

[count number of elements from InputStream without binding](https://github.com/json-iterator/java-json-benchmark/blob/master/src/main/java/com/github/fabienrenaud/jjb/stream/UsersStreamDeserializer.java#L352)

| parser    | score |
| ---       | ---   |
| [javaxjson] | 113.544 ±  6.153  ops/s |
| [jackson]   | 199.957 ±  7.669  ops/s |
| jsoniter (iterator-api) | 274.039 ± 17.785  ops/s |

![java10000](http://jsoniter.com/benchmarks/java10000.png)

# Go Bind API

Different libraries bind data to struct in different ways:

* encoding/json: reflection
* easyjson: go generate
* jsonparser: hand-written data bind
* jsoniter (iterator-api): hand-written data bind
* jsoniter (bind-api): reflection

## Small Payload

[bind small payload of json to struct](https://github.com/json-iterator/go-benchmark/blob/master/src/github.com/json-iterator/go-benchmark/benchmark_small_payload_test.go)

| parser                  | ns/op      | bytes/op | allocs/op   |
| ---                     | ---        | ---      | ---         |
| [encoding/json]           | 3151 ns/op | 480 B/op |	6 allocs/op |
| [easyjson]                | 786 ns/op	 | 64 B/op	| 2 allocs/op |
| [jsonparser]              | 718 ns/op	 | 64 B/op  | 2 allocs/op |
| jsoniter (iterator-api) | 619 ns/op  | 64 B/op  | 2 allocs/op |
| jsoniter (bind-api)     | 844 ns/op  | 256 B/op | 4 allocs/op |

![go-small](http://jsoniter.com/benchmarks/go-small.png)

## Medium Payload

[bind medium payload of json to nested struct](https://github.com/json-iterator/go-benchmark/blob/master/src/github.com/json-iterator/go-benchmark/benchmark_medium_payload_test.go)

| parser                  | ns/op       | bytes/op | allocs/op    |
| ---                     | ---         | ---      | ---          |
| [encoding/json]           | 30531 ns/op	| 808 B/op | 18 allocs/op |
| [easyjson]                | 7731 ns/op  | 248 B/op | 8 allocs/op  |
| [jsonparser]              | 6326 ns/op  | 104 B/op | 4 allocs/op  |
| jsoniter (iterator-api) | 4966 ns/op	| 104 B/op | 4 allocs/op  |
| jsoniter (bind-api)     | 5640 ns/op  | 368 B/op | 14 allocs/op |

![go-medium](http://jsoniter.com/benchmarks/go-medium.png)

# Go Iterator API

## Large Payload

[count number of elements from []byte without binding](https://github.com/json-iterator/go-benchmark/blob/master/src/github.com/json-iterator/go-benchmark/benchmark_large_payload_test.go)

| parser                  | ns/op          | bytes/op   | allocs/op      |
| ---                     | ---            | ---        | ---            |
| [encoding/json]           | 567880 ns/op	 | 79177 B/op | 4918 allocs/op |
| [jsonparser]              | 44660 ns/op	   | 0 B/op	    | 0 allocs/op    |
| jsoniter (iterator-api) | 48737 ns/op    | 0 B/op     | 0 allocs/op    |

![go-large](http://jsoniter.com/benchmarks/go-large.png)

## Large File

[count number of elements from io.Reader without binding](https://github.com/json-iterator/go-benchmark/blob/master/src/github.com/json-iterator/go-benchmark/large_file_test.go)

| parser                  | ns/op           | bytes/op      | allocs/op        |
| ---                     | ---             | ---           | ---              |
| [encoding/json]           | 277265824 ns/op	| 71467156 B/op	| 272476 allocs/op |
| [jsonparser]              | 53586488 ns/op	| 67107204 B/op | 20 allocs/op     |
| jsoniter (iterator-api) | 44817092 ns/op  | 4248 B/op     | 5 allocs/op      |

![go-reader](http://jsoniter.com/benchmarks/go-reader.png)

# Optimization used

## Single pass scan

All parsing is done within one pass directly from byte array stream. Single pass has two level of meaning:

* on the large scale: the iterator api is forward only, you get what you need from current spot. There is no going back.
* on the micro scale: readInt or readString is done in one pass. For example, pass integer is not done, by cut string out then parse string. Instead we use the byte stream to calculate int value directly. readFloat is an exception.

## Minimum allocation

Making copy is avoided at all necessary means. For example, the parser has a internal byte array buffer holding recent byte. When parsing the field name of object, we do not allocate new bytes to hold field name. Instead, if possible, the buffer is reused as slice.

Iterator instance itself keep copy of all kinds of buffer it used, and they can be reused by reset iterator with new input instead of create brand new iterator.

## Pull from stream

The input can be a InputStream or io.Reader, we do not read all bytes out into a big array. Instead the parsing is done in chunks. When we need more, we pull from the stream.

## Take string seriously

String parsing is performance killer if not being handled properly. The trick I learned from [jsonparser] and [dsljson] is taking a fast path for string without escape character. 

For golang, the string is utf-8 bytes based. The fastest way to construct a string is direct cast from []byte to string, if you can make ensure the []byte does not go away or being modified.

For java, the string is utf-16 char based. Parsing utf8 byte stream to utf16 char array is done by the parser directly, instead of using UTF8 charset. The cost of construct string, is simplely a char array copy.

## Schema based

Iterator api is active instead of passive compared to tokenizer api. It does not parse the token out, then if branching. Instead, given the schema, we know exactly what is ahead of us, so we just parse them as what we think it should be. If input disagree, then we raise proper error.

## Skip takes different path

Skip an object or array should take different path learned from [jsonparser]. We do not care about nested field name or so when we are skipping a whole object.

## Table lookup

Some calculation such as what is int value for char '5' can be done ahead of time. 

## Java only optimization

Java parser is dynamically generated using javassist. Because we are actually generating real java source code, the generator can be easily implemented as static annotation processor.

Since the source code is generated, we are not afraid of making it tedious but specific:

```java
public Object decode(java.lang.reflect.Type type, com.jsoniter.Jsoniter iter) {
com.jsoniter.SimpleObject obj = new com.jsoniter.SimpleObject();
for (com.jsoniter.Slice field = iter.readObjectAsSlice(); field != null; field = iter.readObjectAsSlice()) {
switch (field.len) {
case 6: 
if (field.at(0)==102) {
if (field.at(1)==105) {
if (field.at(2)==101) {
if (field.at(3)==108) {
if (field.at(4)==100) {
if (field.at(5)==49) {
obj.field1 = iter.readString();
continue;
}
if (field.at(5)==50) {
obj.field2 = iter.readString();
continue;
}
}
}
}
}
}
break;
}
iter.skip();
}
return obj;
}
```

Also this:

```java
public Object decode(java.lang.reflect.Type type, com.jsoniter.Jsoniter iter) {
if (!iter.readArray()) {
return new int[0];
}
int a1 = iter.readInt();
if (!iter.readArray()) {
return new int[]{ a1 };
}
int a2 = iter.readInt();
if (!iter.readArray()) {
return new int[]{ a1, a2 };
}
int a3 = iter.readInt();
if (!iter.readArray()) {
return new int[]{ a1, a2, a3 };
}
int a4 = (int) iter.readInt();
int[] arr = new int[8];
arr[0] = a1;
arr[1] = a2;
arr[2] = a3;
arr[3] = a4;
int i = 4;
while (iter.readArray()) {
if (i == arr.length) {
int[] newArr = new int[arr.length * 2];
System.arraycopy(arr, 0, newArr, 0, arr.length);
arr = newArr;
}
arr[i++] = iter.readInt();
}
int[] result = new int[i];
System.arraycopy(arr, 0, result, 0, i);
return result;
}
```

## Golang only optimization

binding to object is not using reflect api. Instead the raw pointer is taken out of `interface{}`, then cast to proper pointer type to set value. For example:

```go
*((*int)(ptr)) = iter.ReadInt()
```

Another optimization is we know how many fields are parsing out for a struct, so we can write the field dispatch differently. For no field, we simply skip. For one field, if/else is enough. 2~4 fields switch case. 5 or more fields, we fallback to use map based field dispatching.

Golang version is not using `go generate` as I find it unfriendly to new developer. I might add `go generate` as an option and put same optimization I did for java version. It can be faster. Being able to access the raw pointer, the golang data binding performance is already good enough. As we can see from the benchmark, hand rolled binding code is only a little faster. This case might change, if golang decided to close it's memory layout for direct manipulation, or there is JIT can optimize more if we can get rid of pointer chasing introduced by virtual method.

[jackson]: https://github.com/FasterXML/jackson-databind
[gson]: https://github.com/google/gson
[fastjson]: https://github.com/alibaba/fastjson
[dsljson]: https://github.com/ngs-doo/dsl-json
[javaxjson]: https://jsonp.java.net/
[encoding/json]: https://golang.org/pkg/encoding/json/
[easyjson]: https://github.com/mailru/easyjson
[jsonparser]: https://github.com/buger/jsonparser
