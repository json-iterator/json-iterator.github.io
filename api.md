---
layout: default
title: Json Iterator API
---

* TOC
{:toc}

# API Choices

One thing does not fit all. Jsoniter always put developr friendliness as top priority. No matter how many times faster you claim you can be, what most developer need is a json parser just get the job done. JSON being a weak typed data exchange format, when being parsed in language like Java or Go, it is very often we need to deal with type mismatch or uncertain data structure. Existing solution is not only slow to parse, but put too much work on the shoulder of developer. The motivation to reinvent this wheel is not performance, but to make the parsing as easy as it can be. Benchmarking is just a way to get your attention, sadly.

Jsoniter gives you three api style choices:

* bind-api: which you should stick with most of time
* any-api: when slow is an option
* iterator-api: when maximum flexibility or performance is what needed

And best of all, you can mix them up when parsing one document. Let's see some code

Given this document `{"a": {"b": {"c": "d"}}}`

Parse with Java bind-api + any-api
```java
public class ABC {
    public Any a;
}

Jsoniter iter = Jsoniter.parse("{'a': {'b': {'c': 'd'}}}".replace('\'', '"'));
ABC abc = iter.read(ABC.class);
System.out.println(abc.a.get("b", "c"));
```

Parse with Go bind-api + any-api

```go
type ABC struct {
	a Any
}

iter := ParseString(`{"a": {"b": {"c": "d"}}}`)
abc := &ABC{}
iter.Read(&abc)
fmt.Println(abc.a.Get("b", "c"))
```


Tranditionally, parse json return `Object` or `interface{}`. Then the parer's job is done, and developer's nightmare begins. Being able to mix bind-api and any-api, we can leave parts of the value uncertain, and deal with them later. `Any` has api to extract data out of deeply nested data structure very easily.

Here is another example, `[123, {"name": "taowen", "tags": ["crazy", "hacker"]}]`

Parse with Java iterator-api + bind-api

```java
public class User {
    public int userId;
    public String name;
    public String[] tags;
}

Jsoniter iter = Jsoniter.parse("[123, {'name': 'taowen', 'tags': ['crazy', 'hacker']}]".replace('\'', '"'));
iter.readArray();
int userId = iter.readInt();
iter.readArray();
User user = iter.read(User.class);
user.userId = userId;
iter.readArray(); // end of array
System.out.println(user);
```

Parse with Go iterator-api + bind-api

```go
type User struct {
	userId int
	name string
	tags []string
}

iter := ParseString(`[123, {"name": "taowen", "tags": ["crazy", "hacker"]}]`)
user := User{}
iter.ReadArray()
user.userId = iter.ReadInt()
iter.ReadArray()
iter.Read(&user)
iter.ReadArray() // array end
fmt.Println(user)
```

Use iterator api gives us flexibility to do whatever we want. But binding a big struct is not fun to do manually. By mixing bind-api with iterator-api we can save some effort without compromising performance. Binding is fast. Handwritten binding code is only 10% faster or less.

Here is a list of possible hybrid combination:

* iterator-api => bind-api: use `Read`
* iterator-api => any-api: use `ReadAny`
* bind-api => iterator-api: register type decoder or field decoder
* bind-api => any-api: use `Any` as data type

# Iterator API

## Motivation

Iterator api can extract string/number from stream of bytes without binding them into object. It is highly efficient for data format transformation and number crunching. Jsoniter is not the first json parser with streaming style api, there is standard api like jsonp for java. But I find existing api too hard to use. Here is a comparison:

Given a JSON text like this

```json
{
    "users": [
        {
            "_id": "58451574858913704731",
            "about": "a4KzKZRVvqfBLdnpUWaD",
            "address": "U2YC2AEVn8ab4InRwDmu",
            "age": 27,
            "balance": "I5cZ5vRPmVXW0lhhRzF4",
            "company": "jwLot8sFN1hMdE4EVW7e",
            "email": "30KqJ0oeYXLqhKMLDUg6",
            "eyeColor": "RWXrMsO6xi9cpxPqzJA1",
            "favoriteFruit": "iyOuAekbybTUeDJqkHNI",
            "gender": "ytgB3Kzoejv1FGU6biXu",
            "greeting": "7GXmN2vMLcS2uimxGQgC",
            "guid": "bIqNIywgrzva4d5LfNlm",
            "index": 169390966,
            "isActive": true,
            "latitude": 70.7333712683406,
            "longitude": 16.25873969455544,
            "name": "bvtukpT6dXtqfbObGyBU",
            "phone": "UsxtI7sWGIEGvM2N1Mh0",
            "picture": "8fiyZ2oKapWtH5kXyNDZJjvRS5PGzJGGxDCAk1he1wuhUjxfjtGIh6agQMbjovF10YlqOyzhQPCagBZpW41r6CdrghVfgtpDy7YH",
            "registered": "gJDieuwVu9H7eYmYnZkz",
            "tags": [
                "M2b9n0QrqC",
                "zl6iJcT68v",
                "VRuP4BRWjs",
                "ZY9jXIjTMR"
            ]
        }
    ]
}
```

The example here is written java, because there is no well-known streaming counterpart written in Golang to demonstrate the difference. But the jsoniter api for go and java version is identical. You can get the idea and apply to both versions.

Using jackson library, to count the total number of tags:

```java
public int jackson(JsonParser jParser) throws IOException {
    int totalTagsCount = 0;
    while (jParser.nextToken() != com.fasterxml.jackson.core.JsonToken.END_OBJECT) {
        String fieldname = jParser.getCurrentName();
        if ("users".equals(fieldname)) {
            while (jParser.nextToken() != com.fasterxml.jackson.core.JsonToken.END_ARRAY) {
                totalTagsCount += jacksonUser(jParser);
            }
        }
    }
    return totalTagsCount;
}

private int jacksonUser(JsonParser jParser) throws IOException {
    int totalTagsCount = 0;
    while (jParser.nextToken() != com.fasterxml.jackson.core.JsonToken.END_OBJECT) {
        String fieldname = jParser.getCurrentName();
        switch (fieldname) {
            case "tags":
                jParser.nextToken();
                while (jParser.nextToken() != com.fasterxml.jackson.core.JsonToken.END_ARRAY) {
                    totalTagsCount++;
                }
                break;
            default:
                jParser.nextToken();
                jParser.skipChildren();
        }
    }
    return totalTagsCount;
}
```

Using jsoniter, the iterator api is much more straight forward, there is no concept as Token.

```java
public int jsoniter(Jsoniter iter) throws IOException {
    int totalTagsCount = 0;
    for (String field = iter.readObject(); field != null; field = iter.readObject()) {
        switch (field) {
            case "users":
                while (iter.readArray()) {
                    for (String field2 = iter.readObject(); field2 != null; field2 = iter.readObject()) {
                        switch (field2) {
                            case "tags":
                                while (iter.readArray()) {
                                    iter.skip();
                                    totalTagsCount++;
                                }
                                break;
                            default:
                                iter.skip();
                        }
                    }
                }
                break;
            default:
                iter.skip();
        }
    }
    return totalTagsCount;
}
```

When using jsoniter iterator api, you just need to imagine the document has already been parsed as map or array, and as if we are iterating over the collection.

## Active V.S Passive

The jackson/jsonp api style is:

```java
Token token = parser.next();
if (token == xxx) {
  // do yyy
}
```

The control is passive. You do not know what you will get back from `parser.next()`. The api force you to guard against all kind of cases. Compared to jsoniter:

```java
while (iter.readArray()) {
  // read element
}
```

The api is active instead of passive. I know I must be reading array, if the json is not array, the input must be wrong. The parser should raise proper error message, instead of put the burden of detecting and reporting error on the shoulder of api caller. Let the api caller express its intention before reading next token, makes the code easier to read. It also makes the code more CPU friendly thanks to less branching.

## Do not need to worry about TOKEN

The loop to parse object looks like

```java
for (String field = iter.readObject(); field != null; field = iter.readObject()) {
  switch (field) {
    // deal with fields
  }
}
```

It is as if you are iterating over a hashmap. There is no concept of TOKEN. You do not need to know START_OBJECT, KEY, END_OBJECT. The api is one level higher than tokenizer. Not only easier to use, but much more efficient internally. The parser know the KEY always follow START_OBJECT, and will extract the key out for you in one shot. 

## Schema Free

Iterator api can also work in schema free way. So that in case the input is uncertain, for example some field might be string or int, we can still deal with it.

```java
ValueType valueType = iter.whatIsNext();
if (valueType == ValueType.STRING) {
  return iter.readString();
} else {
  return Integer.toString(iter.readInt());
}
```

It is not strictly schema free here. If the input is not string or int, but a array, `iter.readInt` will throw exception and tell we are expecting a integer, but found a array.

# Bind API

No matter how convenient the iterator api is, iterator is still just iterator. 99% of time, we want to bind the JSON input into object, then process it. Binding JSON to structured object gives the schema info to the parser. With structure, not only easier to underderstand, but also makes the parsing much much faster. 

The api is simple. In Java, you provide the class

```java
Jsoniter iter = Jsoniter.parse("[1,2,3]");
int[] val = iter.read(int[].class);
```

In Go, you provide pointer to your object

```go
iter := ParseString(`[1,2,3]`)
slice := []int{}
iter.Read(&slice)
```

If you want to use generics in Java, use this syntax:

```java
Jsoniter iter = Jsoniter.parse("[1,2,3]");
List<Integer> val = iter.read(new TypeLiteral<ArrayList<Integer>>(){});
```

## Bind callback

The downside of binding is there is always exception. We want the field to be string, but some input put the field as int. Being able to customize the binding is crucial. The answer is callback. Add callback hook in the binding process, then we can use iterator api to take back the control.

Register type decoder to parse element of specific type to use your callback

```java
Jsoniter.registerTypeDecoder(Date.class, new Decoder() {
    @Override
    public Object decode(Type type, Jsoniter iter) throws IOException {
	return new Date(iter.readLong());
    }
});
Jsoniter iter = Jsoniter.parse("1481365190000");
Date date = iter.read(Date.class);
assertEquals(1481365190000L, date.getTime());
Jsoniter.clearDecoders();
```

Register field decoder to special handle the fields chosen

```java
Jsoniter.registerFieldDecoder(SimpleObject.class, "field1", new Decoder(){
    @Override
    public Object decode(Type type, Jsoniter iter) throws IOException {
	return Integer.toString(iter.readInt());
    }
});
Jsoniter iter = Jsoniter.parse("{'field1': 100}".replace('\'', '"'));
SimpleObject myObject = iter.read(SimpleObject.class);
assertEquals("100", myObject.field1);
Jsoniter.clearDecoders();
```


# API List

Static method to create iterator instance

| java | go | comment |
| --- | --- | --- |
| parse | Parse | create iterator from stream |
| parse | ParseBytes | create iterator from byte array |
| parse | ParseString | create iterator from string |
| registerTypeDecoder | RegisterTypeDecoder | add callback to object binding process |
| registerFieldDecoder | RegisterFieldDecoder | add callback to object binding process |

Iterator methods

| java | go | comment |
| --- | --- | --- |
| reset | Reset | reuse iterator instance for new stream |
| reset | ResetBytes | reuse iterator instance for new byte array |
| read | Read | bind next value into object with schema |
| readAny | ReadAny | bind next value into object without schema (will be slow) |
| whatIsNext | WhatIsNext | the value type of next value |
| readNull  | ReadNull  | if next value is null return true |
| skip | Skip | skip the whole next value, even if it is nested object or array |
| readArray | ReadArray | expect more array element, return false if reached end |
| readObject | ReadObject | expect more object field, return null if reached end |
| readString | ReadString | expect "string" |
| readBase64 | ReadBase64 | read base64 encoded string and return byte array |
| readBoolean | ReadBool | expect true of false |
| -  | ReadUint8 | expect number like 123 |
| -  | ReadInt8 | expect number like -123 |
| -  | ReadUint16 | expect number like 123 |
| readShort  | ReadInt16 | expect number like -123 |
| -  | ReadUint32 | expect number like 123 |
| readInt  | ReadInt32 | expect number like -123 |
| -  | ReadUint64 | expect number like 123 |
| readLong  | ReadInt64 | expect number like -123 |
| -  | ReadUint | expect number like 123 |
| -  | ReadInt | expect number like -123 |
| readFloat | ReadFloat32 | expect number like 1.23 |
| readDouble | ReadFloat64 | expect number like 1.23 |
| reportError | ReportError | report error condition |
| -   | iter.Error | golang version use Error field to represent last error |
| currentBuffer | CurrentBuffer | return the parsing buffer as string, for debug purpose |

The api is almost identical. But each version underlying is implemented differently using the most efficient technique for each language. It is not a simple direct translation from one language to another.
