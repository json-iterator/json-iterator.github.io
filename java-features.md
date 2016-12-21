---
layout: default
title: How-to
---

* TOC
{:toc}

# Simple object binding

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

```
return iter.read(TestObject.class);
```

**read into existing object**

```java
TestObject testObject = new TestObject();
return iter.read(testObject);
```

# Constructor binding

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

## hash mode binding

mark the class with jsoniter annotation, then enable annotation support. Or mark the with jackson annotation, then enable jackson annotation support.

```java
JacksonAnnotationSupport.enable();
return iter.read(TestObject.class);
```

the generated code is

```java
public static Object decode_(com.jsoniter.JsonIterator iter) {
    if (iter.readNull()) { com.jsoniter.CodegenAccess.resetExistingObject(iter); return null; }
    int _field1_ = 0;
    int _field2_ = 0;
    if (!com.jsoniter.CodegenAccess.readObjectStart(iter)) { return new com.jsoniter.demo.ConstructorBinding.TestObject(_field1_,_field2_); }
    switch (com.jsoniter.CodegenAccess.readObjectFieldAsHash(iter)) {
        case 1212206434:
            _field1_ = (int)iter.readInt();
            break;
        case 1195428815:
            _field2_ = (int)iter.readInt();
            break;
        default:
            iter.skip();
    }
    while (com.jsoniter.CodegenAccess.nextToken(iter) == ',') {
        switch (com.jsoniter.CodegenAccess.readObjectFieldAsHash(iter)) {
            case 1212206434:
                _field1_ = (int)iter.readInt();
                continue;
            case 1195428815:
                _field2_ = (int)iter.readInt();
                continue;
        }
        iter.skip();
    }
    com.jsoniter.demo.ConstructorBinding.TestObject obj = new com.jsoniter.demo.ConstructorBinding.TestObject(_field1_,_field2_);
    return obj;
}
```

## strict mode binding

same annotation support as hash mode, just enable the strict mode.

```java
JacksonAnnotationSupport.enable();
JsonIterator.enableStrictMode();
return iter.read(TestObject.class);
```

the generated code is:

```java
public static Object decode_(com.jsoniter.JsonIterator iter) {
    if (iter.readNull()) { com.jsoniter.CodegenAccess.resetExistingObject(iter); return null; }
    int _field1_ = 0;
    int _field2_ = 0;
    if (!com.jsoniter.CodegenAccess.readObjectStart(iter)) {
        return new com.jsoniter.demo.ConstructorBinding.TestObject(_field1_,_field2_);
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
                        _field1_ = (int)iter.readInt();
                        continue;
                    }
                    if (field.at(5)==50) {
                        _field2_ = (int)iter.readInt();
                        continue;
                    }
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
                        _field1_ = (int)iter.readInt();
                        continue;
                    }
                    if (field.at(5)==50) {
                        _field2_ = (int)iter.readInt();
                        continue;
                    }
                }
                break;

        }
        iter.skip();
    }
    com.jsoniter.demo.ConstructorBinding.TestObject obj = new com.jsoniter.demo.ConstructorBinding.TestObject(_field1_,_field2_);
    return obj;
}
```

## reflection

same annotation is also supported by reflection decoder.

```java
JacksonAnnotationSupport.enable();
ExtensionManager.registerTypeDecoder(TestObject.class, new ReflectionDecoder(TestObject.class));
return iter.read(TestObject.class);
```

## performance 

| style | ops/s |
| --- | --- | 
| binding + hash mode | 24176429.215 ±  466360.572  ops/s |
| binding + strict mode | 18469307.112 ±  350924.724  ops/s |
| reflection | 6804994.539 ±  178451.154  ops/s |
| jackson + afterburner | 2834318.635 ± 5851771.796  ops/s |

The reflection implementation is not using Map to hold temp variables, which is the major performance win.

# Private field binding

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

## reflection

only reflection mode support private field binding, you have to register type decoder explicitly. Notice that, private field is bindable by default when using reflection decoder, you do not need to mark them as `@JsonProperty` to force the behavior.

```java
ExtensionManager.registerTypeDecoder(TestObject.class, new ReflectionDecoder(TestObject.class));
return iter.read(TestObject.class);
```

## performance

| style | ops/s |
| --- | --- | 
| reflection | 9112485.981 ± 127051.327  ops/s |
| jackson + afterburner | 4547285.044 ± 157520.005  ops/s |
