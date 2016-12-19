---
layout: default
title: How-to
---

* TOC
{:toc}

# Simple Object binding

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

## iterator + switch case

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

## iterator + if/else

```java
TestObject obj = new TestObject();
for (String field = iter.readObject(); field != null; field = iter.readObject()) {
    if (field.equals("field1")) {
        obj.field1 = iter.readInt();
        continue;
    }
    if (field.equals("field2")) {
        obj.field2 = iter.readInt();
        continue;
    }
    iter.skip();
}
return obj;
```

## iterator + if/else + string.intern

```java
TestObject obj = new TestObject();
for (String field = iter.readObject(); field != null; field = iter.readObject()) {
    field = field.intern();
    if (field == "field1") {
        obj.field1 = iter.readInt();
        continue;
    }
    if (field == "field2") {
        obj.field2 = iter.readInt();
        continue;
    }
    iter.skip();
}
return obj;
```

## none strict mode binding

```
return iter.read(TestObject.class);
```

or to be faster, we can save the type literal instance to avoid runtime calculation.

```
TypeLiteral<TestObject> typeLiteral = new TypeLiteral<TestObject>() {}; // reuse this
return iter.read(typeLiteral);
```

the generated code can be printed out using environment variable `$JSONITER_DEBUG=true`

```java
public static Object decode_(com.jsoniter.JsonIterator iter) {
    if (iter.readNull()) { com.jsoniter.CodegenAccess.resetExistingObject(iter); return null; }
    com.jsoniter.demo.SimpleObjectBinding.TestObject obj = (com.jsoniter.CodegenAccess.existingObject(iter) == null ? new com.jsoniter.demo.SimpleObjectBinding.TestObject() : (com.jsoniter.demo.SimpleObjectBinding.TestObject)com.jsoniter.CodegenAccess.resetExistingObject(iter));
    if (!com.jsoniter.CodegenAccess.readObjectStart(iter)) { return obj; }
    switch (com.jsoniter.CodegenAccess.readObjectFieldAsHash(iter)) {
        case 1212206434:
            obj.field1 = (int)iter.readInt();
            break;
        case 1195428815:
            obj.field2 = (int)iter.readInt();
            break;
        default:
            iter.skip();
    }
    while (com.jsoniter.CodegenAccess.nextToken(iter) == ',') {
        switch (com.jsoniter.CodegenAccess.readObjectFieldAsHash(iter)) {
            case 1212206434:
                obj.field1 = (int)iter.readInt();
                continue;
            case 1195428815:
                obj.field2 = (int)iter.readInt();
                continue;
        }
        iter.skip();
    }
    return obj;
}
```

The field binding is done using hash. This implementation is copied from dsljson, but it is not exactly safe when hash collision exists.

## strict mode binding

```java
JsonIterator.enableStrictMode();
return iter.read(TestObject.class);
```

the generated code

```java
public static Object decode_(com.jsoniter.JsonIterator iter) {
    if (iter.readNull()) { com.jsoniter.CodegenAccess.resetExistingObject(iter); return null; }
    com.jsoniter.demo.SimpleObjectBinding.TestObject obj = (com.jsoniter.CodegenAccess.existingObject(iter) == null ? new com.jsoniter.demo.SimpleObjectBinding.TestObject() : (com.jsoniter.demo.SimpleObjectBinding.TestObject)com.jsoniter.CodegenAccess.resetExistingObject(iter));
    if (!com.jsoniter.CodegenAccess.readObjectStart(iter)) {
        return obj;
    }
    com.jsoniter.Slice field = com.jsoniter.CodegenAccess.readObjectFieldAsSlice(iter);
    boolean once = true;
    while (once) {
        once = false;
        switch (field.len) {
            case 6:
                if (field.at(0)==102 &&
                    field.at(1)==105 &&
                    field.at(2)==101 &&
                    field.at(3)==108 &&
                    field.at(4)==100) {
                    if (field.at(5)==49) {
                        obj.field1 = (int)iter.readInt();
                        continue;
                    }
                    if (field.at(5)==50) {
                        obj.field2 = (int)iter.readInt();
                        continue;
                    }
                    continue;
                }
                break;

        }
        iter.skip();
    }
    while (com.jsoniter.CodegenAccess.nextToken(iter) == ',') {
        field = com.jsoniter.CodegenAccess.readObjectFieldAsSlice(iter);
        switch (field.len) {
            case 6:
                if (field.at(0)==102 &&
                    field.at(1)==105 &&
                    field.at(2)==101 &&
                    field.at(3)==108 &&
                    field.at(4)==100) {
                    if (field.at(5)==49) {
                        obj.field1 = (int)iter.readInt();
                        continue;
                    }
                    if (field.at(5)==50) {
                        obj.field2 = (int)iter.readInt();
                        continue;
                    }
                    continue;
                }
                break;

        }
        iter.skip();
    }
    return obj;
}
```

This implementation can ensure the field is exactly match, but slower.

## reflection

Jsoniter also support using reflection to bind object. It will be necessary to bind to private/protected members. To force the reflection decoder being used to need to:

```java
ExtensionManager.registerTypeDecoder(TestObject.class, new ReflectionDecoder(TestObject.class));
return iter.read(TestObject.class);
```

## read into existing object

```java
TestObject testObject = new TestObject();
return iter.read(testObject);
```

to avoid runtime calculation of decoder cache key, we can use the saved type literal

```java
TypeLiteral<TestObject> typeLiteral = new TypeLiteral<TestObject>() {}; // can be reused
TestObject testObject = new TestObject(); // can be reused
return iter.read(typeLiteral, testObject);
```

Allowing to reuse existing object is convenient for stream processing. Same object can be reused again an again for a long array of objects.

## performance

| style | ops/s |
| --- | --- | 
| iterator + switch case | 10161124.096 ±  86811.453  ops/s |
| iterator + if/else |  10566399.972 ± 219004.245  ops/s |
| iterator + if/else + string.intern | 4091671.061 ±  59660.899  ops/s |
| binding + strict mode | 19654075.094 ± 734231.072  ops/s |
| binding + none strict mode | 25177037.170 ± 379426.831  ops/s |
| reflection | 7017747.269 ± 653401.038  ops/s |
| dsljson | 10536139.221 ± 204085.931  ops/s |
| jackson + afterburner | 4912109.137 ±  81390.018  ops/s | 
| jackson | 4346835.527 ± 163832.141  ops/s | 
| fastjson | 2467929.303 ± 101384.609  ops/s |

Advice on iterator: do not use string.intern, it will not be faster. If/else might be slightly faster than switch/case, but swtich case is much more readable.

None strict mode binding is the fastest, strict mode comes second, and iterator api is slower. The difference is iterator api need to constructor a string object for the field. Strict mode binding is using a slice object reusing the underlying byte array. None strict mode binding only need to compute a hash code.

