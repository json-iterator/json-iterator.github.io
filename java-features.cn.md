---
layout: default
title: Jsoniter Features (Java Version)
---

* TOC
{:toc}

# 超简单的 api

## 反序列化

```java
Any obj = JsonIterator.deserialize("[1,2,3]");
System.out.println(obj.get(2));
int[] array = JsonIterator.deserialize("[1,2,3]", int[].class);
System.out.println(array[2]);
```

只有一个静态方法，没法更简单了

## 序列化
```java
System.out.println(JsonStream.serialize(new int[]{1,2,3}));
```

就一个静态方法，真没法更简单了

# 给不同的任务选择不同的 api

Jsoniter 有三个不同的 api 用于不同的场合：

* iterator-api：用于处理超大的输入
* bind-api：日常最经常使用的对象绑定
* any-api：lazy 解析大对象，具有 PHP Array 一般的使用体验

而且你还能自由混搭出6个组合

## bind-api + any-api

```java
public class ABC {
    public Any a; // lazy parsed
}

JsonIterator iter = JsonIterator.parse("{'a': {'b': {'c': 'd'}}}".replace('\'', '"'));
ABC abc = iter.read(ABC.class);
System.out.println(abc.a.get("b", "c"));
```

对于某些字段不想立刻被解析，可以用 Any 保存起来，用到的时候再延迟解析

## iterator-api + bind-api

```java
public class User {
    public int userId;
    public String name;
    public String[] tags;
}

JsonIterator iter = JsonIterator.parse("[123, {'name': 'taowen', 'tags': ['crazy', 'hacker']}]".replace('\'', '"'));
iter.readArray();
int userId = iter.readInt();
iter.readArray();
User user = iter.read(User.class);
user.userId = userId;
iter.readArray(); // end of array
System.out.println(user);
```

使用 iterator 做流式解析的时候，一个个绑定字段很麻烦，可以局部使用一下 bind-api

## any-api + bind-api

```java
String input = "{'numbers': ['1', '2', ['3', '4']]}".replace('\'', '"');
String[] array = JsonIterator.deserialize(input).get("numbers", 2).to(String[].class);
```

从 `Any` 里面抽取出来了值，然后还能用 bind-api 绑定到对象上

## 总共 6 种组合！

* iterator-api => bind-api: JsonIterator.read
* iterator-api => any-api: JsonIterator.readAny
* bind-api => iterator-api: register type decoder or property decoder
* bind-api => any-api: use `Any` as data type
* any-api => bind-api: Any.to(class)
* any-api => iterator-api: JsonIterator.parse(any)

# 极致性能需要代码生成

缺省的编解码的方式是反射。如果用 javassist 实现动态代码生成的话性能可以成倍提升。它可以给对应的输入的 class 生成定制化的高效代码。然而动态代码生成在某些平台上不可用，所以静态代码生成的用法也是支持的。

* 反射：默认选项，零依赖
* 动态代码生成：需要 javassist 库
* 静态代码生成：麻烦一点，但是也可以这么用

## 动态代码生成

把这个依赖添加到你的项目里

```xml
<dependency>
    <groupId>org.javassist</groupId>
    <artifactId>javassist</artifactId>
    <version>3.21.0-GA</version>
</dependency>
```

然后把模式设置为动态代码生成

```java
JsonIterator.setMode(DecodingMode.DYNAMIC_MODE_AND_MATCH_FIELD_WITH_HASH);
JsonStream.setMode(EncodingMode.DYNAMIC_MODE);
JsoniterAnnotationSupport.enable();
```

所有的功能应该都能正常工作的，而且要快很多

## 反射
反射可以给具体的某个 class 启用，也可以全局开启。比如，对于这个 class

```java
public class TestObject {
    private int field1;
    private int field2;
}
```

为了把值绑定到私有成员上，我们必须对这个类启用反射

```java
JsoniterSpi.registerTypeDecoder(TestObject.class, ReflectionDecoderFactory.create(TestObject.class));
return iter.read(TestObject.class); // will use reflection
```

或者我们也可以把默认的模式设置为反射

```java
JsonIterator.setMode(DecodingMode.REFLECTION_MODE);
JsoniterAnnotationSupport.enable();
```

所有的特性在反射模式下都是支持的，只是比代码生成的要慢一点。但是还是比其他的解决方案快很多，这里是一个简单的对象多字段绑定的性能评测：

| parser | ops/s |
| --- | --- |
| jackson + afterburner | 6632322.908 ±  248913.699  ops/s |
| jsoniter + reflection | 11484306.001 ±  139780.870  ops/s |
| jsoniter + codegen | 31486700.029 ±  373069.642  ops/s |

## 静态代码生成

如果你想要最好的性能，但是你使用的平台又无法支持动态代码生成的时候，你可以选择静态代码生成。要启用静态代码生成，需要完成三件事情：

* 提前定义哪些 class 是需要编解码的
* 把代码生成加入到 build 的过程中，比如 maven
* 把模式切换为 static

首先我们来定义哪些class是需要编解码的

```java
public class DemoCodegenConfig implements CodegenConfig {

    @Override
    public void setup() {
        // register custom decoder or extensions before codegen
        // so that we doing codegen, we know in which case, we need to callback
        JsoniterSpi.registerFieldDecoder(User.class, "score", new Decoder.IntDecoder() {
            @Override
            public int decodeInt(JsonIterator iter) throws IOException {
                return Integer.valueOf(iter.readString());
            }
        });
    }

    @Override
    public TypeLiteral[] whatToCodegen() {
        return new TypeLiteral[]{
                // generic types, need to use this syntax
                new TypeLiteral<List<Integer>>() {
                },
                new TypeLiteral<Map<String, Object>>() {
                },
                // array
                TypeLiteral.create(int[].class),
                // object
                TypeLiteral.create(User.class)
        };
    }
}
```

然后我们在 maven 中添加代码生成的调用：

```
<plugin>
<groupId>org.codehaus.mojo</groupId>
<artifactId>exec-maven-plugin</artifactId>
<version>1.5.0</version>
<executions>
    <execution>
	<id>static-codegen</id>
	<phase>compile</phase>
	<goals>
	    <goal>exec</goal>
	</goals>
	<configuration>
	    <executable>java</executable>
	    <workingDirectory>${project.build.sourceDirectory}</workingDirectory>
	    <arguments>
		<argument>-classpath</argument>
		<classpath/>
		<argument>com.jsoniter.StaticCodeGenerator</argument>
		<argument>com.jsoniter.demo.DemoCodegenConfig</argument>
	    </arguments>
	</configuration>
    </execution>
</executions>
</plugin>
```

产生的代码会被写到你项目的 `src/main/java` 目录，作为你的代码的一部分。最后把模式切换一下

```java
JsonStream.setMode(EncodingMode.STATIC_MODE); 
JsonIterator.setMode(DecodingMode.STATIC_MODE); // set mode before using
JsoniterAnnotationSupport.enable();
new JsonIterator().read(...
```

把模式设置为 static 之后，动态代码生成就不会被自动触发了。如果对应的类没有预先生成的编解码代码，异常会被抛出。

# 对象绑定的多种姿势

Java程序员是矫情的。相比 golang 的纯真朴素，java 里给一个对象赋值的方式不要太多了。Jsoniter 入乡随俗，不会强制只支持 field 绑定的方式的。以下是所有的对象绑定可能：

## 公有 field 绑定

给定这样的文档

```json
{"field1":100,"field2":101}
```

绑定到这个 class 上

```java
public class TestObject {
    public int field1;
    public int field2;
}
```

bind-api 好用，但是不是唯一的选择。你也可以使用 iterator-api 来手工完成绑定

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

如果用 bind-api，就可以简化为一行代码：

```java
return iter.read(TestObject.class);
```

**read into existing object**

```java
TestObject testObject = new TestObject();
return iter.read(testObject);
```

Jsoniter 深圳允许你复用已有的对象，把值直接绑定上去。当你需要反复地绑定对象的时候，这可以节省内存分配的时间。

## 构造函数绑定

绑定这个文档

```json
{"field1":100,"field2":101}
```

到 class 上

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

需要把类添加上 jsoniter 的 annotation，然后开始 annotation 的支持。如果你已经在使用 jackson 的 annotation，也可以开启 JacksonAnnotationSupport 的兼容模式。必须把参数用 `@JsonProperty` 标记，因为旧的 java 版本无法通过反射获得参数名。

```java
JacksonAnnotationSupport.enable(); // use JsoniterAnnotationSupport if you are not using Jackson
return iter.read(TestObject.class);
```

`@JsonCreator` 不仅仅支持构造函数，静态函数充当工厂方法也是可以的。

## Setter 绑定

绑定这个文档

```json
{"field1":100,"field2":101}
```

绑定到使用 setter 的这个 class 上

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

这个写法是自动支持的。甚至如果 setter 有多个参数也是可以的（严格意义上来说，这就不是setter了），但是需要 annotation 的支持。

```java
public static class TestObject {
    private int field1;
    private int field2;

    @JsonWrapper
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

## 私有成员绑定

绑定这个文档

```json
{"field1":100,"field2":101}
```

到这个类上

```java
public class TestObject {
    private int field1;
    private int field2;
}
```

**reflection**

只有反射模式才支持私有成员的绑定。注意当使用反射的时候，无需标记 `@JsonProperty`，所有字段默认都会被序列化和反序列化，除非使用 `@JsonIgnore` 排除掉。

```java
JsoniterSpi.registerTypeDecoder(TestObject.class, ReflectionDecoderFactory.create(TestObject.class));
return iter.read(TestObject.class);
```
