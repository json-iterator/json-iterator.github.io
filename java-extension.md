---
layout: default
title: How to extend
---

# Bind-api extension

Bind-api is the most complex api. Iterating the json input, while binding to a object graph has a infinity ways to customize. 
If the tool does not provide the flexibility to extend, then the user is forced to bind the object graph twice.
The goal of jsoniter is not to be the fastest JSON parser, but to provide the flexibility I always wanted.
The general direction to provide extensiblity, is to give the user a callback, instead of just a list of feature flags or annotation.
All kinds of annotation support can be implemented using the callback. Also the callback should control the codegen if possible, not 
slowing down the runtime execution.

# Decoder interface

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
