---
layout: default
title: How to extend
---

# Migration from gson

Jsoniter support gson annotation, migrating from gson to jsoniter should be easy.

```
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

```
GsonCompatibilityMode config = new GsonCompatibilityMode.Builder()
        .excludeFieldsWithoutExposeAnnotation()
        .build();
TestObject obj = JsonIterator.deserialize(config, "{\"field1\":\"hello\"}", TestObject.class);
```
