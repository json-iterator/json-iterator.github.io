---
layout: default
title: Go JSON 使用小技巧
---

* TOC
{:toc}

有的时候上游传过来的字段是string类型的，但是我们却想用变成数字来使用。
本来用一个`json:",string"` 就可以支持了，如果不知道golang的这些小技巧，就要大费周章了。

参考文章：http://attilaolah.eu/2014/09/10/json-and-struct-composition-in-go/

# 临时忽略struct字段

```golang
type User struct {
    Email    string `json:"email"`
    Password string `json:"password"`
    // many more fields…
}
```

临时忽略掉Password字段

```golang
json.Marshal(struct {
    *User
    Password bool `json:"password,omitempty"`
}{
    User: user,
})
```

# 临时添加额外的字段

```golang
type User struct {
    Email    string `json:"email"`
    Password string `json:"password"`
    // many more fields…
}
```

临时忽略掉Password字段，并且添加token字段

```golang
json.Marshal(struct {
    *User
    Token    string `json:"token"`
    Password bool `json:"password,omitempty"`
}{
    User: user,
    Token: token,
})
```

# 临时粘合两个struct

```golang
type BlogPost struct {
    URL   string `json:"url"`
    Title string `json:"title"`
}

type Analytics struct {
    Visitors  int `json:"visitors"`
    PageViews int `json:"page_views"`
}

json.Marshal(struct{
    *BlogPost
    *Analytics
}{post, analytics})
```

# 一个json切分成两个struct

```golang
json.Unmarshal([]byte(`{
  "url": "attila@attilaolah.eu",
  "title": "Attila's Blog",
  "visitors": 6,
  "page_views": 14
}`), &struct {
  *BlogPost
  *Analytics
}{&post, &analytics})
```

# 临时改名struct的字段

```golang
type CacheItem struct {
    Key    string `json:"key"`
    MaxAge int    `json:"cacheAge"`
    Value  Value  `json:"cacheValue"`
}

json.Marshal(struct{
    *CacheItem

    // Omit bad keys
    OmitMaxAge omit `json:"cacheAge,omitempty"`
    OmitValue  omit `json:"cacheValue,omitempty"`

    // Add nice keys
    MaxAge int    `json:"max_age"`
    Value  *Value `json:"value"`
}{
    CacheItem: item,

    // Set the int by value:
    MaxAge: item.MaxAge,

    // Set the nested struct by reference, avoid making a copy:
    Value: &item.Value,
})
```

# 用字符串传递数字

```golang
type TestObject struct {
	Field1 int    `json:",string"`
}
```

这个对应的json是 `{"Field1": "100"}`

如果json是 `{"Field1": 100}` 则会报错

# 容忍字符串和数字互转

如果你使用的是jsoniter，可以启动模糊模式来支持 PHP 传递过来的 JSON。

```golang
import "github.com/json-iterator/go/extra"

extra.RegisterFuzzyDecoders()
```

这样就可以处理字符串和数字类型不对的问题了。比如

```golang
var val string
jsoniter.UnmarshalFromString(`100`, &val)
```

又比如

```golang
var val float32
jsoniter.UnmarshalFromString(`"1.23"`, &val)
```

# 容忍空数组作为对象

PHP另外一个令人崩溃的地方是，如果 PHP array是空的时候，序列化出来是`[]`。但是不为空的时候，序列化出来的是`{"key":"value"}`。
我们需要把 `[]` 当成 `{}` 处理。

如果你使用的是jsoniter，可以启动模糊模式来支持 PHP 传递过来的 JSON。

```golang
import "github.com/json-iterator/go/extra"

extra.RegisterFuzzyDecoders()
```

这样就可以支持了

```golang
var val map[string]interface{}
jsoniter.UnmarshalFromString(`[]`, &val)
```

# 使用 MarshalJSON支持time.Time

golang 是默认不支持 time.Time 序列化成 JSON 的。如果要支持，可以自定义 time.Time 类型


```golang
type timeImplementedMarshaler time.Time

func (obj timeImplementedMarshaler) MarshalJSON() ([]byte, error) {
	seconds := time.Time(obj).Unix()
	return []byte(strconv.FormatInt(seconds, 10)), nil
}
```

序列化的时候会调用 MarshalJSON

```golang
type TestObject struct {
	Field timeImplementedMarshaler
}
should := require.New(t)
val := timeImplementedMarshaler(time.Unix(123, 0))
obj := TestObject{val}
bytes, err := jsoniter.Marshal(obj)
should.Nil(err)
should.Equal(`{"Field":123}`, string(bytes))
```

# 使用 RegisterTypeEncoder支持time.Time
# 使用 MarshalText支持非字符串作为key的map
# 使用 json.RawMessage
# 使用 json.Number
# 统一更改字段的命名风格
# 使用私有的字段
