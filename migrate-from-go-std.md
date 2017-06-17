---
layout: default
title: Migrate from go standard libary
---

# Drop-in replacement

```
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

# 100% Compatibility

By default, jsoniter do not sort the map keys like standard libary. If you want 100% compatibility, use it like this

```
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
* use `jsoniter.RegisterTypeEncoder` or `jsoniter.RegisterTypeDecoder` instead of `MarshalJSON`. `Marshaler` or `Unmarshaler` interface will do more copying than necessary. 


