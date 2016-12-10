---
layout: default
title: Json Iterator API
---

* TOC
{:toc}

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

The api is active instead of passive. I know I must be reading array, if the json is not array, it is the file wrong, not my fault. The parser should raise proper error message, instead of put the burden of detecting and reporting error on the shoulder of api caller. Let the api caller express its intention before reading next token, not only makes the code easier to read, also makes the code CPU friendly because less branching.

## readObject

The loop to parse object looks like

```java
for (String field = iter.readObject(); field != null; field = iter.readObject()) {
  switch (field) {
    // deal with fields
  }
}
```

It is as if you are iterating over a hashmap. There is no concept of TOKEN. You do not need to know START_OBJECT, KEY, END_OBJECT. The api is one level higher than tokenizer. Not only easier to use, but much more efficient internally. The parser know the KEY always follow START_OBJECT, and will extract the key out for you in one shot. 

## whatIsNext

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

# Integration

# API List

| java | go | comment |
| --- | --- | --- |
| parse | Parse | create iterator from stream |
| parse | ParseBytes | create iterator from byte array |
| parse | ParseString | create iterator from string |
| reset | Reset | reuse iterator instance for new stream |
| reset | ResetBytes | reuse iterator instance for new byte array |
| whatIsNext | WhatIsNext | the value type of next value |
| readNull  | ReadNull  | if next value is null return true |
| skip | Skip | skip the whole next value, even if it is object or array |
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
