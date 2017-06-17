---
layout: default
title: Migrate from golang standard libary
---

# Everything just works

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
b, err := json.Marshal(group)
```

add `import "github.com/json-iterator/go"` and repalce `json.Marshal` with `jsoniter.Marshal`. 
`Unmarshal`, `NewEncoder`, `NewDecoder` they all works.


