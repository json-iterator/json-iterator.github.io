---
layout: default
title: Go JSON tips
---

* TOC
{:toc}

There are times we are passed with a field of string type, but we wish it to be int. 
If we know `json:",string"`, then it should be a easy. 
Otherwise, it will take some serious time to do the conversion.

Rerference: http://attilaolah.eu/2014/09/10/json-and-struct-composition-in-go/

# Ad-hoc ignore some field

```golang
type User struct {
    Email    string `json:"email"`
    Password string `json:"password"`
    // many more fields…
}
```

to ignore `Password` field

```golang
json.Marshal(struct {
    *User
    Password bool `json:"password,omitempty"`
}{
    User: user,
})
```

# Ad-hoc add extra field

```golang
type User struct {
    Email    string `json:"email"`
    Password string `json:"password"`
    // many more fields…
}
```

Ignore the `Password` field and add new `Token` field

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

# Ad-hoc combine two structs

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

# Ad-hoc split one json into two

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

# Ad-hoc rename field

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

# Pass numbers as string

```golang
type TestObject struct {
	Field1 int    `json:",string"`
}
```

The corresponding json should be `{"Field1": "100"}`

For `{"Field1": 100}`, there will be error

# String/number fuzzy conversion

If you are using jsoniter, enable fuzzy decoders can make life easier working with PHP.

```golang
import "github.com/json-iterator/go/extra"

extra.RegisterFuzzyDecoders()
```

Then, no matter the input is string or number, or the output is number or string, jsoniter will convert it for you. For example:

```golang
var val string
jsoniter.UnmarshalFromString(`100`, &val)
```

And

```golang
var val float32
jsoniter.UnmarshalFromString(`"1.23"`, &val)
```

# [] as object

Another heart breaking "feature" of PHP is that, when array is empty, it is encoded as `[]` instead of `{}`.

If you are using jsoniter, enable fuzzy decoders will also fix this.

```golang
import "github.com/json-iterator/go/extra"

extra.RegisterFuzzyDecoders()
```

Then we can deal with `[]`

```golang
var val map[string]interface{}
jsoniter.UnmarshalFromString(`[]`, &val)
```

# Customize time.Time with MarshalJSON

The defeault implementation will encode time.Time as string. If we want to represent time in other format, we have to define a new type
for time.Time, with MarshalJSON.

```golang
type timeImplementedMarshaler time.Time

func (obj timeImplementedMarshaler) MarshalJSON() ([]byte, error) {
	seconds := time.Time(obj).Unix()
	return []byte(strconv.FormatInt(seconds, 10)), nil
}
```

Marshal will invoke MarshalJSON on types defined them.

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

# Use RegisterTypeEncoder to customize time.Time

Unlike the standard library, jsoniter allow you to customize types defined by other guys. For example, we can use epoch int64 to encode a time.Time.

```golang
import "github.com/json-iterator/go/extra"

extra.RegisterTimeAsInt64Codec(time.Microsecond)
output, err := jsoniter.Marshal(time.Unix(1, 1002))
should.Equal("1000001", string(output))
```

If you want to do it yourself, reference `RegisterTimeAsInt64Codec` for more information.

# Non string key map

Although JSON standard requires the key of map to be string, golang support more types as long as they defined MarshalText(). For example:

```golang
f, _, _ := big.ParseFloat("1", 10, 64, big.ToZero)
val := map[*big.Float]string{f: "2"}
str, err := MarshalToString(val)
should.Equal(`{"1":"2"}`, str)
```

Given `big.Float` is a type with MarshalText()

# Use json.RawMessage

Some part of JSON might be lacking a uniform schema, we can keep the original JSON as it is.

```golang
type TestObject struct {
	Field1 string
	Field2 json.RawMessage
}
var data TestObject
json.Unmarshal([]byte(`{"field1": "hello", "field2": [1,2,3]}`), &data)
should.Equal(` [1,2,3]`, string(data.Field2))
```

# Use json.Number

By default, number decoded into interface{} will be of type float64. If the input is large, the precision loss might be a problem.
So we can `UseNumber()` to represent number as `json.Number`.

```golang
decoder1 := json.NewDecoder(bytes.NewBufferString(`123`))
decoder1.UseNumber()
var obj1 interface{}
decoder1.Decode(&obj1)
should.Equal(json.Number("123"), obj1)
```

jsoniter support this usage. And it extended the support to `Unmarshal` as will.

```golang
json := Config{UseNumber:true}.Froze()
var obj interface{}
json.UnmarshalFromString("123", &obj)
should.Equal(json.Number("123"), obj)
```

# Field naming strategy

Most of the time, we want to different field names in JSON. We can use field tag:

```golang
output, err := jsoniter.Marshal(struct {
	UserName      string `json:"user_name"`
	FirstLanguage string `json:"first_language"`
}{
	UserName:      "taowen",
	FirstLanguage: "Chinese",
})
should.Equal(`{"user_name":"taowen","first_language":"Chinese"}`, string(output))
```

However, it is tedious. If you are using jsoniter, we can change all fields in one line.

```golang
import "github.com/json-iterator/go/extra"

extra.SetNamingStrategy(LowerCaseWithUnderscores)
output, err := jsoniter.Marshal(struct {
	UserName      string
	FirstLanguage string
}{
	UserName:      "taowen",
	FirstLanguage: "Chinese",
})
should.Nil(err)
should.Equal(`{"user_name":"taowen","first_language":"Chinese"}`, string(output))
```

# Private fields

The standard library requries all fields appear in JSON to be public. If you use jsoniter, `SupportPrivateFields()` will remove this restriction.

```golang
import "github.com/json-iterator/go/extra"

extra.SupportPrivateFields()
type TestObject struct {
	field1 string
}
obj := TestObject{}
jsoniter.UnmarshalFromString(`{"field1":"Hello"}`, &obj)
should.Equal("Hello", obj.field1)
```
