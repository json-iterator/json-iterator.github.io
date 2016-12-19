---
layout: default
title: How to extend
---

# Bind-api extension

Bind-api is the most complex api. Iterating the json input, while binding to a object graph has infinite ways to customize. 
If the tool does not provide the flexibility to extend, then the user is forced to bind the object graph twice.
The goal of jsoniter is not to be the fastest JSON parser, but to provide the flexibility I always wanted.
The general direction to provide extensiblity, is to give the user a callback, instead of just a list of feature flags or annotation.
All kinds of annotation support can be implemented using the callback. Also the callback should control the codegen if possible, not 
slowing down the runtime execution.

# Customize type decoding

```java
public interface Decoder {
    /**
     * Customized decoder to read values from iterator
     *
     * @param iter the iterator instance
     * @return the value to set
     * @throws IOException when reading from iterator triggered error
     */
    Object decode(JsonIterator iter) throws IOException;
}
```

The unique extensiblity of jsoniter is coming from the iterator-api. The simplest form of bind-api callback is `Decoder`, which takes the current `iterator` instance pointing to the next value to read, and read the object for the binding. For example

```java
JsonIterator.registerTypeDecoder(Date.class, new Decoder() {
    @Override
    public Object decode(final JsonIterator iter) throws IOException {
        return new Date(iter.readLong());
    }
});
```

when a `Date` class to read, the registered decoder will be called. 

```java
JsonIterator iter = JsonIterator.parse("1481365190000");
iter.read(Date.class); // Sat Dec 10 18:19:50 CST 2016
```

Generic types can also be customized by `TypeLiteral`. Here is an example:

```java
TypeLiteral<List<Integer>> listOfIntType = new TypeLiteral<List<Integer>>() {
};
JsonIterator.registerTypeDecoder(listOfIntType, new Decoder() {
    @Override
    public Object decode(JsonIterator iter) throws IOException {
        ArrayList<Integer> list = new ArrayList<Integer>();
        for (char c : iter.readString().toCharArray()) {
            list.add(c - '0');
        }
        return list;
    }
});
JsonIterator iter = JsonIterator.parse("\"1481365190000\"");
List<Integer> listOfInt = iter.read(listOfIntType);
System.out.println(listOfInt); // [1, 4, 8, 1, 3, 6, 5, 1, 9, 0, 0, 0, 0]
```

# Customize field decoding

Sometimes, we can not just customize the type, all different occurence of the type will need different encoding. So we can precisely control individual field decoding. The callback interface is still `Decoder`, we just need to register it to field instead of type.

```java
public class TestObject1 {
    public String field1;
}

JsonIterator.registerFieldDecoder(TestObject1.class, "field1", new Decoder() {

    @Override
    public Object decode(JsonIterator iter) throws IOException {
        return Integer.toString(iter.readInt());
    }
});
JsonIterator iter = JsonIterator.parse("{'field1': 100}".replace('\'', '"'));
TestObject1 myObject = iter.read(TestObject1.class);
assertEquals("100", myObject.field1);
```

For field decoding, there is one thing special: primitive types. If the field is of type like `int` or `long`, we do not want to return `Object` from `Decoder` then unbox it to primitive types. The decoder registered by user should not be forced to go through this process. And `IntDecoder` is what we need:

```java
public class TestObject2 {
    public int field1;
}

JsonIterator.registerFieldDecoder(TestObject2.class, "field1", new Decoder.IntDecoder() {

    @Override
    public int decodeInt(JsonIterator iter) throws IOException {
        return Integer.valueOf(iter.readString());
    }
});
JsonIterator iter = JsonIterator.parse("{'field1': '100'}".replace('\'', '"'));
TestObject2 myObject = iter.read(TestObject2.class);
assertEquals(100, myObject.field1);
```

# Rename field

Besides controlling how to decode a value, changing which JSON field bind to which object field is a very common requirement. Instead of only provide a `@JsonProperty` to change the mapping, jsoniter provide a much more powerful solution, a callback interface `Extension`. There are many things a extension can customize, here we focus on field renaming.

```java
public class TestObject4 {
    public int field1;
}

ExtensionManager.registerExtension(new EmptyExtension() {
    @Override
    public void updateClassDescriptor(ClassDescriptor desc) {
        if (desc.clazz != TestObject4.class) {
            return;
        }
        for (Binding field : desc.allDecoderBindings()) {
            if (field.name.equals("field1")) {
                field.fromNames = new String[]{"field_1", "Field1"};
            }
        }
    }
});
JsonIterator iter = JsonIterator.parse("{'field_1': 100}".replace('\'', '"'));
TestObject4 myObject1 = iter.read(TestObject4.class);
assertEquals(100, myObject1.field1);
```

The callback `updateClassDescriptor` change the data source or decoder for a field. The `fromNames` is a `String[]`, with following options:

* null: do not customize this field
* empty array: disable this field binding. works like `@JsonIgnore`
* one or many: you can map multiple possible json field to one object field

The input is a concept called `Binding`, which not only represent field, but also constructor parameter and setter parameter. It contains the necessary information about what is binding, and you can customize how to bind it:

```java
public class Binding {
    // input
    public Class clazz;
    public String name;
    public Type valueType;
    public TypeLiteral valueTypeLiteral;
    public Annotation[] annotations;
    // output
    public String[] fromNames; // for decoder
    public String[] toNames; // for encoder
    public Decoder decoder;
    public Encoder encoder;
    public boolean failOnMissing;
    public boolean failOnPresent;
    // optional
    public Field field;
    public int idx;
}
```

So we can also customize the field decoder using `Extension` to update bindings. Actually type decoder and field decoder callback is just a shortcut. The `Extension` interface can do all the job.

# Your own annotation support

It is very common to support `@JsonProperty` to rename field. Jsoniter also comes with built-in support for annotation. But by registering your own extension, you can implement them very easily.

```java
@Target({ElementType.ANNOTATION_TYPE, ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface JsonProperty {
    String USE_DEFAULT_NAME = "";
    String value() default USE_DEFAULT_NAME;
}

public class MyAnnotationSupport extends EmptyExtension {

    @Override
    public void updateClassDescriptor(ClassDescriptor desc) {
        for (Binding field : desc.allDecoderBindings()) {
            JsonIgnore jsonIgnore = field.getAnnotation(JsonIgnore.class);
            if (jsonIgnore != null) {
                field.fromNames = new String[0];
            }
            JsonProperty jsonProperty = field.getAnnotation(JsonProperty.class);
            if (jsonProperty != null) {
                String alternativeField = jsonProperty.value();
                if (!alternativeField.isEmpty()) {
                    field.fromNames = new String[]{alternativeField};
                }
            }
        }
    }
}
```

Then you can annotate

```java
public static class AnnotatedObject {
    @JsonProperty("field-1")
    public int field1;
}

JsonIterator iter = JsonIterator.parse("{'field-1': 100}".replace('\'', '"'));
AnnotatedObject obj = iter.read(AnnotatedObject.class);
assertEquals(100, obj.field1);
```

If you do not want to write annotation support yourself, you can use built-in `com.jsoniter.annotation.JsoniterAnnotationSupport`.

# No default constructor? No problem!

Jsoniter can work with class without default constructor. The extension can customize the constructor to be used for specific class. The constructor can also be a static method, instead of real constructor.

```java
public class ConstructorDescriptor {
    /**
     * set to null if use constructor
     * otherwise use static method
     */
    public String staticMethodName;
    // optional
    public Constructor ctor;
    // optional
    public Method staticFactory;

    /**
     * the parameters to call constructor or static method
     */
    public List<Binding> parameters = new ArrayList<Binding>();
}
```

Using this extension point, `@JsonCreator` annotation support has been implemented. You can mark it on constructor:

```java
public static class NoDefaultCtor {
    private int field1;

    @JsonCreator
    public NoDefaultCtor(@JsonProperty("field1") int field1) {
        this.field1 = field1;
    }
}
```

Or, you can mark it on static method:

```java
public static class StaticFactory {

    private int field1;

    private StaticFactory() {
    }

    @JsonCreator
    public static StaticFactory createObject(@JsonProperty("field1") int field1) {
        StaticFactory obj = new StaticFactory();
        obj.field1 = field1;
        return obj;
    }
}
```

To enable jsoniter annotation support `JsoniterAnnotationSupport.enable()` must be called first

# Setter support

The binding is done in this sequence

```
1. create instance using constructor
2. bind fields
3. call setters
```

By default, methods like `setName(val)` will be caleld with field value from `name`. You can choose to use other kinds of methods as setters. For example

```java
public static class WithSetter {

    private int field1;
    private int field2;

    @JsonSetter
    public void initialize(@JsonProperty("field1") int field1, @JsonProperty("field2") int field2) {
        this.field1 = field1;
        this.field2 = field2;
    }
}
```

This annotation support is implemented by `Extension` as well. The callback is:

```java
public class SetterDescriptor {
    /**
     * which method to call to set value
     */
    public String methodName;

    /**
     * the parameters to bind
     */
    public List<Binding> parameters = new ArrayList<Binding>();

    // optional
    public Method method;
}
```

The setter parameters, constructor parameters and fields are all just bindings. Those bindings can be updated by extension, to change data source and decoder.

# Extension & Iterator

```java
public interface Extension {
    /**
     * Customize type decoding
     *
     * @param cacheKey
     * @param type change how to decode the type
     * @return null, if no special customization needed
     */
    Decoder createDecoder(String cacheKey, Type type);

    /**
     * Update binding is done for the class
     *
     * @param desc binding information
     */
    void updateClassDescriptor(ClassDescriptor desc);
}
```

To summary it up, you have these options:

* control how to decode one type, include generic type with different type arguments
* the constructor to use, and the parameter binding of constructor
* the setters to use, and the parameter binding of setters
* all bindings (fields, setter parameters, constructor parameters) can change data source (even map to multiple fields) and change decoder

Also the decoder interface is powered with iterator-api to iterate on the lowest level. I believe this should provide all the flexibility you will ever need. If you are still not on board, here is another shot. 

There is a feature flag of jackson called `ACCEPT_EMPTY_ARRAY_AS_NULL_OBJECT`. The intention is to decode input like `[]` as null, as PHP might treat empty object as empty array. To support this feature, we can register a extension

```java
ExtensionManager.registerExtension(new EmptyExtension() {
    @Override
    public Decoder createDecoder(final String cacheKey, final Type type) {
        if (cacheKey.endsWith(".original")) {
            // avoid infinite loop
            return null;
        }
        if (type != Date.class) {
            return null;
        }
        return new Decoder() {
            @Override
            public Object decode(JsonIterator iter1) throws IOException {
                if (iter1.whatIsNext() == ValueType.ARRAY) {
                    if (iter1.readArray()) {
                        // none empty array
                        throw iter1.reportError("decode [] as null", "only empty array is expected");
                    } else {
                        return null;
                    }
                } else {
                    // just use original decoder
                    TypeLiteral typeLiteral = new TypeLiteral(type, cacheKey + ".original",
                            TypeLiteral.generateEncoderCacheKey(type));
                    return iter1.read(typeLiteral);
                }
            }
        };
    }
});
JsonIterator iter = JsonIterator.parse("[]");
assertNull(iter.read(Date.class));
```

Using iterator we can detect if input is `[]`, and act accordingly. If input is not array, we can even fallback to default behavior. It works like servlet filter, if one decoder can not handle it, it can fallback to the next one.


