---
layout: default
title: Migrate from gson
---

# Migrate from gson

Jsoniter support gson annotation, migrating from gson to jsoniter should be easy.

```java
public static class TestObject {
    @Expose(deserialize = false)
    public String field1;
}

Gson gson = new GsonBuilder()
        .excludeFieldsWithoutExposeAnnotation()
        .create();
TestObject obj = gson.fromJson("{\"field1\":\"hello\"}", TestObject.class);
```

will be

```java
GsonCompatibilityMode config = new GsonCompatibilityMode.Builder()
        .excludeFieldsWithoutExposeAnnotation()
        .build();
TestObject obj = JsonIterator.deserialize(config, "{\"field1\":\"hello\"}", TestObject.class);
```

serialization also works

```java
public static class TestObject {
    public String field1 = "hello";
}

Gson gson = new GsonBuilder()
        .setFieldNamingPolicy(FieldNamingPolicy.UPPER_CAMEL_CASE)
        .create();
String output = gson.toJson(new TestObject());
assertEquals("{\"Field1\":\"hello\"}", output);
```

will be

```java
GsonCompatibilityMode config = new GsonCompatibilityMode.Builder()
        .setFieldNamingPolicy(FieldNamingPolicy.UPPER_CAMEL_CASE)
        .build();
String output = JsonStream.serialize(config, new TestObject());
assertEquals("{\"Field1\":\"hello\"}", output);
```
