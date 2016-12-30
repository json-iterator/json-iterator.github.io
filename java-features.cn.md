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
