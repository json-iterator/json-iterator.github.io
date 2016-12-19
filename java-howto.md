---
layout: default
title: How-to
---

# Simple Object binding

bind the json document 

```json
{"field1":100,"field2":101}
```

to the class

```java
public static class TestObject {
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

## performance

| style | ops/s |
| --- | --- | 
| iterator + switch case | 10161124.096 ±  86811.453  ops/s |
| iterator if/else |  10566399.972 ± 219004.245  ops/s |
| iterator if/else + string.intern | 4091671.061 ±  59660.899  ops/s |

Do not use string.intern, it will not be faster. If/else might be slightly faster than switch/case, but swtich case is much more readable.
