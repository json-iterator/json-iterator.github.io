---
layout: default
title: How to extend
---


# Bind-api extension

Bind-api is the most complex api. Iterating the json input, while binding to a object graph has infinite ways to customize. 
If the tool does not provide the flexibility to extend, then the user is forced to bind the object graph twice.
The goal of jsoniter is not to be the fastest JSON parser, but to provide the flexibility I always wanted.
The general direction to provide extensiblity, is to give the user a callback, instead of just a list of feature flags or annotation.
All kinds of annotation support can be implemented using the callback. Also the callback should control the codegen if possible, not 
slowing down the runtime execution.

# Customize type decoding

```go
RegisterTypeDecoder("time.Time", func(ptr unsafe.Pointer, iter *Iterator) {
  t, err := time.ParseInLocation("2006-01-02 15:04:05", iter.ReadString(), time.UTC)
  if err != nil {
    iter.Error = err
    return
  }
  *((*time.Time)(ptr)) = t
})
defer ClearDecoders()
val := time.Time{}
err := Unmarshal([]byte(`"2016-12-05 08:43:28"`), &val)
if err != nil {
  t.Fatal(err)
}
year, month, day := val.Date()
if year != 2016 || month != 12 || day != 5 {
  t.Fatal(val)
}
```

The callback will be called for the registered type. The first argument is a pointer to the instance, the second argument is the iterator pointint to the next value to read.

# Customize field decoding

We can also control how the deconding is done at the field level. For example if the input is int, and we want to bind it to a string typed field. 

```go
type Tom struct {
	field1 string
}
```

and the input `{"field1": 100}`. We need to register a field decoder:

```go
RegisterFieldDecoder("jsoniter.Tom", "field1", func(ptr unsafe.Pointer, iter *Iterator) {
  *((*string)(ptr)) = strconv.Itoa(iter.ReadInt())
})
defer ClearDecoders()
tom := Tom{}
err := Unmarshal([]byte(`{"field1": 100}`), &tom)
if err != nil {
  t.Fatal(err)
}
```

It is nearly the same as type decoder, just add a argument to specify the field name.

# Customize field mapping

When the field name does not match the json input, we can customize it by callback. The callback will be provided with `reflect.StructField`, which has tags for the field.

```go
RegisterExtension(func(type_ reflect.Type, field *reflect.StructField) ([]string, DecoderFunc) {
  if (type_.String() == "jsoniter.Tom" && field.Name == "field1") {
    return []string{"field-1"}, nil
  }
  return nil, nil
})
tom := Tom{}
err := Unmarshal([]byte(`{"field-1": "100"}`), &tom)
if err != nil {
  t.Fatal(err)
}
if tom.field1 != "100" {
  t.Fatal(tom.field1)
}
```

The extension definition is 

```go
type ExtensionFunc func(type_ reflect.Type, field *reflect.StructField) ([]string, DecoderFunc)
```

* type is the type being decoded
* field is the field being decoded. this could be nil, if customizing the type itself
* first return value is alternative field names for the field, only used when field is not nil
* second return value is the decoder to decode the type or just the field, depending on field is nil or not


