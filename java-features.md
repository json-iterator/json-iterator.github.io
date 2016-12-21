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

```java
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

**binding**

mark the class with jsoniter annotation, then enable annotation support. Or if you are already using jackson, you can still use jackson annotation, just turn on JacksonAnnotationSupport. It is important to mark `@JsonProperty` with field name, as old java version can not get parameter name by reflection.

```java
JacksonAnnotationSupport.enable(); // use JsoniterAnnotationSupport if you are not using Jackson
return iter.read(TestObject.class);
```

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
