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

```
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

JsonIterator.registerExtension(new EmptyExtension() {
    @Override
    public boolean updateBinding(Binding field) {
        if (field.clazz == TestObject4.class && field.name.equals("field1")) {
            field.fromNames = new String[]{"field_1", "Field1"};
            return true;
        }
        return false;
    }
});
JsonIterator iter = JsonIterator.parse("{'field_1': 100}".replace('\'', '"'));
TestObject4 myObject1 = iter.read(TestObject4.class);
assertEquals(100, myObject1.field1);
```

The callback `updateBinding` change the data source or decoder for a field. The `fromNames` is a `String[]`, with following options:

* null: do not customize this field
* empty array: disable this field binding. works like `@JsonIgnore`
* one or many: you can map multiple possible json field to one object field

The input is a concept called `Binding`, which not only represent field, but also constructor parameter and setter parameter. It contains the necessary information about what is binding, and you can customize how to bind it:

```
public class Binding {
    // input
    public Class clazz;
    public String name;
    public Type valueType;
    public Annotation[] annotations;
    // output
    public String[] fromNames;
    public Decoder decoder;
}
```

So we can also customize the field decoder using `Extension` to update bindings. Actually type decoder and field decoder callback is just a shortcut. The `Extension` interface can do all the job.
