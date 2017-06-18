---
layout: default
title: Migrate from go standard libary
---

# Drop-in replacement

```golang
type ColorGroup struct {
	ID     int
	Name   string
	Colors []string
}
group := ColorGroup{
	ID:     1,
	Name:   "Reds",
	Colors: []string{"Crimson", "Red", "Ruby", "Maroon"},
}
b, err := jsoniter.Marshal(group)
```

Add `import "github.com/json-iterator/go"` and repalce `json.Marshal` with `jsoniter.Marshal`. Then the code should behave exactly the same, just much faster. Unlike easyjson or other json libaries, jsoniter does not rely on static code generation.

`Unmarshal`, `NewEncoder`, `NewDecoder` they all works. Existing types implemented `Marshaler` or `Unmarshaler` interface will also work. Map with non-string key also work. Yes, everything just works.

# One line style

You can parse json in one line, without defining any struct.

```golang
val := []byte(`{"ID":1,"Name":"Reds","Colors":["Crimson","Red","Ruby","Maroon"]}`)
jsoniter.Get(val, "Colors", 0).ToString()
```

`func Get(data []byte, path ...interface{}) Any` takes interface{} as path. If string, it will lookup json map. 
If int, it will lookup json array. If `'*'`, it will map to each element of array or each key of map.

It will be faster than parsing into `map[string]interface{}` and much easier to read data out.

# 100% Compatibility

By default, jsoniter do not sort the map keys like standard libary. If you want 100% compatibility, use it like this

```golang
m := map[string]interface{}{
	"3": 3,
	"1": 1,
	"2": 2,
}
json := jsoniter.ConfigCompatibleWithStandardLibrary
b, err := json.Marshal(m)
```

# Best performance

The default performance is already several times faster than the standard library. If you want to have absolutely best performance, you can do following things

* use jsoniter.ConfigFastest, this will marshal the float with 6 digits precision (lossy), which is significantly faster
* reuse the underlying Stream or Iterator instance. `jsoniter.ConfigFastest.BorrowIterator` or `jsoniter.ConfigFastest.BorrowStream`. Just remember to return them when done.
* use `jsoniter.RegisterTypeEncoder` or `jsoniter.RegisterTypeDecoder` instead of defining `MarshalJSON` or `UnmarshalJSON`. `Marshaler` or `Unmarshaler` interface will do more copying than necessary. 


