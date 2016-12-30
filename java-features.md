---
layout: default
title: Jsoniter Features (Java Version)
---

* TOC
{:toc}

# Very simple api

## deserialize

```java
Any obj = JsonIterator.deserialize("[1,2,3]");
System.out.println(obj.get(2));
int[] array = JsonIterator.deserialize("[1,2,3]", int[].class);
System.out.println(array[2]);
```

just one static method, can not be more simple

## serialize

```java
System.out.println(JsonStream.serialize(new int[]{1,2,3}));
```

just one static method, can not be more simple


# Right api for the right job

Jsoniter come with three api for different parsing job:

* iterator-api: when input is large
* bind-api: bind value to object
* any-api: lazy parsing large object

And you can mix and match

## bind-api + any-api

```java
public class ABC {
    public Any a; // lazy parsed
}

JsonIterator iter = JsonIterator.parse("{'a': {'b': {'c': 'd'}}}".replace('\'', '"'));
ABC abc = iter.read(ABC.class);
System.out.println(abc.a.get("b", "c"));
```

when certain field do not want to bind eagerly, we can parse them later.

## iterator-api + bind-api

```java
public class User {
    public int userId;
    public String name;
    public String[] tags;
}

JsonIterator iter = JsonIterator.parse("[123, {'name': 'taowen', 'tags': ['crazy', 'hacker']}]".replace('\'', '"'));
iter.readArray();
int userId = iter.readInt();
iter.readArray();
User user = iter.read(User.class);
user.userId = userId;
iter.readArray(); // end of array
System.out.println(user);
```

when using itrator, but do not want to bind field by field

## any-api + bind-api

```java
String input = "{'numbers': ['1', '2', ['3', '4']]}".replace('\'', '"');
String[] array = JsonIterator.deserialize(input).get("numbers", 2).to(String[].class);
```

after you extracted value from `Any`, then you can bind it to object using bind-api

## total 6 combinations!

* iterator-api => bind-api: JsonIterator.read
* iterator-api => any-api: JsonIterator.readAny
* bind-api => iterator-api: register type decoder or property decoder
* bind-api => any-api: use `Any` as data type
* any-api => bind-api: Any.to(class)
* any-api => iterator-api: JsonIterator.parse(any)

# Performance is optional

The default decoding/encoding mode is reflection. We can improve the performance by dynamically generated decoder/encoder class using javassist. It will generate the most efficient code for the given class of input. However, dynamically code generation is not available in all platforms, so static code generation is also provided as an option.

* reflection: the default
* dynamic code generation: require javassist library dependency
* static code generation: also an option

## Dynamic code generation

add this dependency to your project

```xml
<dependency>
    <groupId>org.javassist</groupId>
    <artifactId>javassist</artifactId>
    <version>3.21.0-GA</version>
</dependency>
```

then set the mode to dynamic code generation

```java
JsonIterator.setMode(DecodingMode.DYNAMIC_MODE_AND_MATCH_FIELD_WITH_HASH);
JsonStream.setMode(EncodingMode.DYNAMIC_MODE);
JsoniterAnnotationSupport.enable();
```

everything should still works, just much much faster

## Reflection

reflection can be enabled per-class, or globally. For example, for the class

```java
public class TestObject {
    private int field1;
    private int field2;
}
```

to bind to private field, we have to enable reflection for this type

```java
JsoniterSpi.registerTypeDecoder(TestObject.class, ReflectionDecoderFactory.create(TestObject.class));
return iter.read(TestObject.class); // will use reflection
```

Or, we can set the default decoding mode to reflection

```java
JsonIterator.setMode(DecodingMode.REFLECTION_MODE);
JsoniterAnnotationSupport.enable();
```

All features is supported using the reflection mode, just slower, but still faster than existing solutions. Here is a quick benchmark of simple object field binding:

| parser | ops/s |
| --- | --- |
| jackson + afterburner | 6632322.908 ±  248913.699  ops/s |
| jsoniter + reflection | 11484306.001 ±  139780.870  ops/s |
| jsoniter + codegen | 31486700.029 ±  373069.642  ops/s |

## Static code generation

When you want to have maximum performance, and the platform you use can not support dynamic code generation, then you can try static codegen. To enable static codegen, 3 things need to be done:

* define what class need to be decoded or encoded
* add code generation to maven build process
* switch mode to static codegen when decoding or encoding


```java
public class DemoCodegenConfig implements CodegenConfig {

    @Override
    public void setup() {
        // register custom decoder or extensions before codegen
        // so that we doing codegen, we know in which case, we need to callback
        JsoniterSpi.registerFieldDecoder(User.class, "score", new Decoder.IntDecoder() {
            @Override
            public int decodeInt(JsonIterator iter) throws IOException {
                return Integer.valueOf(iter.readString());
            }
        });
    }

    @Override
    public TypeLiteral[] whatToCodegen() {
        return new TypeLiteral[]{
                // generic types, need to use this syntax
                new TypeLiteral<List<Integer>>() {
                },
                new TypeLiteral<Map<String, Object>>() {
                },
                // array
                TypeLiteral.create(int[].class),
                // object
                TypeLiteral.create(User.class)
        };
    }
}
```

then add code generation to maven build:

```
<plugin>
<groupId>org.codehaus.mojo</groupId>
<artifactId>exec-maven-plugin</artifactId>
<version>1.5.0</version>
<executions>
    <execution>
	<id>static-codegen</id>
	<phase>compile</phase>
	<goals>
	    <goal>exec</goal>
	</goals>
	<configuration>
	    <executable>java</executable>
	    <workingDirectory>${project.build.sourceDirectory}</workingDirectory>
	    <arguments>
		<argument>-classpath</argument>
		<classpath/>
		<argument>com.jsoniter.StaticCodeGenerator</argument>
		<argument>com.jsoniter.demo.DemoCodegenConfig</argument>
	    </arguments>
	</configuration>
    </execution>
</executions>
</plugin>
```

the generated code will be written out to `src/main/java` folder of your project. The final step is to switch mode

```java
JsonIterator.setMode(DecodingMode.STATIC_MODE); // set mode before using
JsoniterAnnotationSupport.enable();
new JsonIterator().read(...
```

by setting the mode to static, dynamic code generation will not happen if the class to decode/encode does not have corresponding decoder/encoder, instead exception will be thrown.

# Object binding styles

binding to object can be done using a couple of ways. Jsoniter will not force you to make fields public

## Public field binding

bind the json document 

```json
{"field1":100,"field2":101}
```

to the class

```java
public class TestObject {
    public int field1;
    public int field2;
}
```

bind-api is not your only option. You can also use iterator-api to do the binding manually. 

**iterator + switch case**

```java
TestObject obj = new TestObject();
for (String field = iter.readObject(); field != null; field = iter.readObject()) {
    switch (field) {
        case "field1":
            obj.field1 = iter.readInt();
            continue;
        case "field2":
            obj.field2 = iter.readInt();
            continue;
        default:
            iter.skip();
    }
}
return obj;
```

**binding**

```java
return iter.read(TestObject.class);
```

**read into existing object**

```java
TestObject testObject = new TestObject();
return iter.read(testObject);
```

Jsoniter allow reusing existing object. When you need to bind one object repeatedly, this can save the memory allocation cost.

## Constructor binding

bind the json document 

```json
{"field1":100,"field2":101}
```

to the class

```java
public class TestObject {
    private int field1;
    private int field2;

    @JsonCreator
    public TestObject(
            @JsonProperty("field1") int field1,
            @JsonProperty("field2") int field2) {
        this.field1 = field1;
        this.field2 = field2;
    }
}
```

**binding**

mark the class with jsoniter annotation, then enable annotation support. Or if you are already using jackson, you can still use jackson annotation, just turn on JacksonAnnotationSupport. It is important to mark `@JsonProperty` with field name, as old java version can not get parameter name by reflection.

```java
JacksonAnnotationSupport.enable(); // use JsoniterAnnotationSupport if you are not using Jackson
return iter.read(TestObject.class);
```

Not only constructor binding is supported, static method working as factory method is also supported. Just mark it with `@JsonCreator`.

## Setter Binding

bind the json document 

```json
{"field1":100,"field2":101}
```

to the class, using setter

```java
public static class TestObject {
    private int field1;
    private int field2;

    public void setField1(int field1) {
        this.field1 = field1;
    }

    public void setField2(int field2) {
        this.field2 = field2;
    }
}
```

this is automatically supported. Multi parameter setter is also supported, but need to use annotation support.

```java
public static class TestObject {
    private int field1;
    private int field2;

    @JsonWrapper
    public void initialize(
            @JsonProperty("field1") int field1,
            @JsonProperty("field2") int field2) {
        this.field1 = field1;
        this.field2 = field2;
    }
}
```

```java
JsoniterAnnotationSupport.enable();
return iter.read(TestObject.class);
```

## Private field binding

bind the json document 

```json
{"field1":100,"field2":101}
```

to the class

```java
public class TestObject {
    private int field1;
    private int field2;
}
```

**reflection**

only reflection mode support private field binding, you can register type decoder explicitly. Notice that, private field is bindable by default when using reflection decoder, you do not need to mark them as `@JsonProperty` to force the behavior. If you do not want the private field to be bind, you have to use `@JsonIgnore` to ignore them.

```java
JsoniterSpi.registerTypeDecoder(TestObject.class, ReflectionDecoderFactory.create(TestObject.class));
return iter.read(TestObject.class);
```

# Wrapper & Unwrapper

## Wrapper

say you have this kind of object graph

```java
public class Name {
    private final String firstName;
    private final String lastName;

    public Name(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    public String getFirstName() {
        return firstName;
    }

    public String getLastName() {
        return lastName;
    }
}

public class User {
    private Name name;
    public int score;

    public Name getName() {
        return name;
    }

    @JsonWrapper
    public void setName(@JsonProperty("firstName") String firstName, @JsonProperty("lastName") String lastName) {
        name = new Name(firstName, lastName);
    }
}
```

the `Name` is a wrapper class. and the input is flat:

```json
{"firstName": "tao", "lastName": "wen", "score": 100}
```

we can see there is no direct mapping between the json and object. this is when `@JsonWrapper` is helpful. essentially, it is binding to function parameters.

## Unwrapper

For the same object graph, if we want to serialize it back, by default the output looks like:

```java
JsonStream.serialize(user)
// {"score":100,"name":{"firstName":"tao","lastName":"wen"}}
```

Using `@JsonUnwrapper` we can control how to write out its member:

```java
public class User {
    private Name name;
    public int score;

    @JsonIgnore
    public Name getName() {
        return name;
    }

    @JsonUnwrapper
    public void writeName(JsonStream stream) throws IOException {
        stream.writeObjectField("firstName");
        stream.writeVal(name.getFirstName());
        stream.writeMore();
        stream.writeObjectField("lastName");
        stream.writeVal(name.getLastName());
    }

    @JsonWrapper
    public void setName(@JsonProperty("firstName") String firstName, @JsonProperty("lastName") String lastName) {
        System.out.println(firstName);
        name = new Name(firstName, lastName);
    }
}
```

then the output will be

```java
JsonStream.serialize(user)
// {"score":100,"firstName":"tao","lastName":"wen"}
```

# Validation

It is very common to json decode to a object, then business logic validation is applied. The json can have more fields or less fields than the object. it will be very hard for the user to catch field not set condidtion, as the information is lost after the binding. Jsoniter is one of the few json parsers has implemneted required property tracking. You can tell if a int field is 0, because there is no input, or input is 0.

## Missing field

```java
public static class TestObject {
    @JsonProperty(required = true)
    public int field1;
    @JsonProperty(required = true)
    public int field2;
    @JsonProperty(required = true)
    public int field3;
}
```

if `field1` is not present in the input json, exception will be thrown.

```java
JsoniterAnnotationSupport.enable();
JsonIterator iter = JsonIterator.parse("{'field2':101}".replace('\'', '"'));
return iter.read(TestObject.class);
```

the output is 

```
com.jsoniter.JsonException: missing mandatory fields: [field1, field3]

	at decoder.com.jsoniter.demo.MissingField.TestObject.decode_(TestObject.java)
	at decoder.com.jsoniter.demo.MissingField.TestObject.decode(TestObject.java)
	at com.jsoniter.JsonIterator.read(JsonIterator.java:339)
	at com.jsoniter.demo.MissingField.withJsoniter(MissingField.java:85)
	at com.jsoniter.demo.MissingField.test(MissingField.java:60)
```

If you do not want to throw exception, you can add `@JsonMissingProperties` to catch those fields.

```java
public static class TestObject {
    @JsonProperty(required = true)
    public int field1;
    @JsonProperty(required = true)
    public int field2;
    @JsonProperty(required = true)
    public int field3;
    @JsonMissingProperties
    public List<String> missingFields; // will be [field1, field3]
}
```

## Fail on unknown properties

```java
@JsonObject(asExtraForUnknownProperties = true)
public static class TestObject2 {
    public int field1;
    public int field2;
}
```

using `failOnUnknown` we can detect if the input has extra fields. Enable the annotation support:

```java
JsoniterAnnotationSupport.enable();
JsonItertor iter = JsonIterator.parse("{'field1':101,'field2':101,'field3':101}".replace('\'', '"').getBytes());
return iter.read(TestObject2.class);
```

the error message looks like 

```
com.jsoniter.JsonException: extra property: field3
```

If you do not want to throw exception, you can add `@JsonExtraProperties` to catch them:

```java
@JsonObject(asExtraForUnknownProperties = true)
public static class TestObject2 {
    public int field1;
    public int field2;
    @JsonExtraProperties
    public Map<String, Any> extra; // will contain field3
}
```

## Exclude whitelist properties

```java
@JsonObject(asExtraForUnknownProperties = true, unknownPropertiesWhitelist = {"field2"})
public static class TestObject3 {
    public int field1;
}
```

if the input is `{"field1":100,"field2":101}`, it shall pass. if the input is 

```json
{"field1":100,"field2":101,"field3":102}
```

exception will still be thrown. `com.jsoniter.JsonException: unknown property: field3`

## Only fail on blacklist properties

To ensure certain fields should never appear from the input. we can set the blacklist

```java
@JsonObject(unknownPropertiesBlacklist = {"field3"})
public static class TestObject4 {
    public int field1;
}
```

given the input

```json
{"field1":100,"field2":101,"field3":102}
```

will throw exception

```
com.jsoniter.JsonException: extra property: field3
```

# Collection and generics

## Collection

generic collection need to be specified using `TypeLiteral`

| feature | sample |
| --- | --- |
| array | `new JsonIterator().read("[1,2,3,4]", int[].class)` |
| list | `new JsonIterator().read("[1,2,3,4]", new TypeLiteral<List<Integer>>(){})` |
| set | `new JsonIterator().read("[1,2,3,4]", new TypeLiteral<Set<Integer>>(){})` |
| linked list | `new JsonIterator().read("[1,2,3,4]", new TypeLiteral<LinkedList<Integer>>(){})` |
| list of object | `new JsonIterator().read("[1,2,3,4]", List.class)` |
| map | `new JsonIterator().read("{\"a\":1,\"b\":2}", new TypeLiteral<Map<String, Integer>>(){})` |

collection can also be decoded using the iterator-api, for example, given the input

```json
[1,2,3,4]
```

if we use binding to sum, the code looks like

```java
int[] arr = iter.read(int[].class);
int total = 0;
for (int i = 0; i < arr.length; i++) {
    total += arr[i];
}
return total;
```

using iterator, we can get rid of the itermediate array

```java
int total = 0;
while (iter.readArray()) {
    total += iter.readInt();
}
return total;
```

the performance difference is huge

| parser | ops/s |
| --- | --- |
| jackson | 4419446.858 ±  88015.833  ops/s |
| jsoniter + binding | 15061063.604 ± 453904.401  ops/s |
| jsoniter + iterator | 26425709.524 ± 333111.069  ops/s |

## Generic field

If the generic type is specified when defining the field, its component type can be recognized

```java
public class TestObject {
    public List<Integer> values;
}

new JsonIterator().read("{\"values\":[1,2,3]}", TestObject.class);
```

Generic field type variable can also be filled by the sub-class

```java
public class SuperClass<E> {
    public List<E> values;
}

public class SubClass extends SuperClass<Integer> {
}

new JsonIterator().read("{\"values\":[1,2,3]}", SubClass.class);
```

The generic type is instantiated with proper type arguments by one of the three ways:

* TypeLiteral
* Sub-class
* Direct field definition

If the type is not specified, `Object.class` will be used instead.

## Interface type

If the type binding to is a interface, we have to choose a proper implementation to instantiate the object. The defaults are:

| interface | impl |
| --- | --- |
| List | ArrayList |
| Set | HashSet |
| Map | HashMap |

Other interface types will be specified by the user. 

```java
JsoniterSpi.registerTypeImplementation(MyInterface.class, MyObject.class);
```

This will use `MyObject.class` to create object wheree `MyInterface.class` instance is needed.

# Lazy is an option

It is tedious to design a class to describe the schema of data. Jsoniter allow you to decode the json as an `Any` object, and start using it right away. The experience is very similar to PHP json_decode.

## Lazy is not slow

given a larget json document

```json
[
// many many more
  {
    "_id": "58659f976f045f9c69c53efb",
    "index": 4,
    "guid": "3cbaef3d-25ab-48d0-8807-1974f6aad336",
    "isActive": true,
    "balance": "$1,854.63",
    "picture": "http://placehold.it/32x32",
    "age": 26,
    "eyeColor": "brown",
    "name": "Briggs Larson",
    "gender": "male",
    "company": "ZOXY",
    "email": "briggslarson@zoxy.com",
    "phone": "+1 (807) 588-3350",
    "address": "994 Nichols Avenue, Allamuchy, Guam, 3824",
    "about": "Sunt velit ullamco consequat velit ad nisi in sint qui qui ut eiusmod eu. Et ut aliqua mollit cupidatat et proident tempor do est enim exercitation amet aliquip. Non exercitation proident do duis non ullamco do esse dolore in occaecat. Magna ea labore aliqua laborum ad amet est incididunt et quis cillum nulla. Adipisicing veniam nisi esse officia dolor labore. Proident fugiat consequat ullamco fugiat. Est et adipisicing eiusmod excepteur deserunt pariatur aute commodo dolore occaecat veniam dolore.\r\n",
    "registered": "2014-07-21T03:28:39 -08:00",
    "latitude": -59.741245,
    "longitude": -9.657004,
    "friends": [
      {
        "id": 0,
        "name": "Herminia Mcknight"
      },
      {
        "id": 1,
        "name": "Leann Harding"
      },
      {
        "id": 2,
        "name": "Marisol Sykes"
      }
    ],
    "tags": [
      "ea",
      "velit",
      "sunt",
      "fugiat",
      "do",
      "Lorem",
      "nostrud"
    ],
    "greeting": "Hello, Briggs Larson! You have 3 unread messages.",
    "favoriteFruit": "apple"
  }
// many many more
]
```

if we use traditional way to parse it, it will be list of hash map

```java
List users = (List) iter.read();
int total = 0;
for (Object userObj : users) {
    Map user = (Map) userObj;
    List friends = (List) user.get("friends");
    total += friends.size();
}
return total;
```

using `Any`, we can lazy parse it:

```java
Any users = iter.readAny();
int total = 0;
for (Any user : users) { // array parsing triggered
    total += user.getValue("friends").size(); // object parsing triggered
}
return total;
```

the laziness can be verified by benchmark:

| parsing style | ops/s |
| --- | --- |
| eager | 48510.942 ± 3891.295  ops/s |
| lazy | 65088.956 ± 5026.304  ops/s |

## Any is easy

Any can get value directly from nested structure. Any can read value in the type you want, regardless the actual type in the json.

```java
String input = "{'numbers': ['1', '2', ['3', '4']]}".replace('\'', '"');
int value = JsonIterator.deserialize(input).toInt("numbers", 2, 0); // value is 3, converted from string
```

you can detect if the value is present or not, by a single get

```java
String input = "{'numbers': ['1', '2', ['3', '4']]}".replace('\'', '"');
Any any = JsonIterator.deserialize(input);
Any found = any.get("num", 100); // found is null, so we know it is missing from json
```

# Stream parsing

when dealing with large json input, we might want to process them in a streaming way. I found existing solution to be cumbersome to use, so jsoniter (json iterator) is invented.

given we want to parse this document, and count tags

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

## Tokenizer is cumbersome

https://jsonp.java.net/ is awful, it does not even have a api to skip a whole value, so I will skip that. Jackson is the only streaming parser I can find. Here is the implementation:

```java
public int calc(JsonParser jParser) throws IOException {
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

looks not bad actually. the problem is the api is a tokenizer. You will get a token and decide what to do with it. If you did not handle the all cases for the token, then when the input is not what you want, it is very hard to debug.

## Iterator to rescue

Same logic implemented using jsoniter iterator-api:

```java
public int calc(JsonIterator iter) throws IOException {
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

if we change the input a little bit

```json
{
    "users": [
        {
            // ...
            "tags": {
                "tag1": "M2b9n0QrqC",
                "tag2": "zl6iJcT68v",
                "tag3": "VRuP4BRWjs",
                "tag4": "ZY9jXIjTMR"
            }
        }
    ]
}
```

the error message is comprehensible:

```
com.jsoniter.JsonException: readArray: expect [ or , or n or ], but found: {, head: 1010, peek:  "tags": {, buf: {
```

