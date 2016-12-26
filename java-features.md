---
layout: default
title: How-to
---

* TOC
{:toc}

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

    @JsonSetter
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

only reflection mode support private field binding, you have to register type decoder explicitly. Notice that, private field is bindable by default when using reflection decoder, you do not need to mark them as `@JsonProperty` to force the behavior.

```java
ExtensionManager.registerTypeDecoder(TestObject.class, ReflectionDecoderFactory.create(TestObject.class));
return iter.read(TestObject.class);
```

# Fields matching

The json can have more fields or less fields than the object. it will be very hard for the user to catch field not set condidtion, as the information is lost after the binding. 

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

## Fail on unknown properties

```java
@JsonUnknownProperties(failOnUnkown = true)
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
com.jsoniter.JsonException: unknown property: field3
```

## Exclude whitelist properties

```java
@JsonUnknownProperties(failOnUnkown = true, whitelist = {"field2"})
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
@JsonUnknownProperties(blacklist = {"field3"})
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
com.jsoniter.JsonException: found should not present field: field3
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
ExtensionManager.registerTypeImplementation(MyInterface.class, MyObject.class);
```

This will use `MyObject.class` to create object wheree `MyInterface.class` instance is needed.

# Decoding mode

by default, the decoding is done by dynamically generated decoder class. It will generate the most efficient code for the given class of input. However, dynamically code generation is not available in all platforms. So we provide more variety of options:

* dynamic codegen
* static codegen
* reflection

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
ExtensionManager.registerTypeDecoder(TestObject.class, ReflectionDecoderFactory.create(TestObject.class));
return iter.read(TestObject.class); // will use reflection
```

Or, we can set the default decoding mode to reflection

```java
JsonIterator.setMode(DecodingMode.REFLECTION_MODE);
```

All features is supported using the reflection mode, just slower, but still faster than existing solutions. Here is a quick benchmark of simple object field binding:

| parser | ops/s |
| --- | --- |
| jackson + afterburner | 6632322.908 ±  248913.699  ops/s |
| jsoniter + reflection | 11484306.001 ±  139780.870  ops/s |
| jsoniter + codegen | 31486700.029 ±  373069.642  ops/s |
