---
layout: default
title: Go JSON 使用小技巧
---

* TOC
{:toc}

有的时候上游传过来的字段是string类型的，但是我们却想用变成数字来使用。
本来用一个`json:",string"` 就可以支持了，如果不知道golang的这些小技巧，就要大费周章了。

# 临时忽略struct字段
# 临时粘合两个struct
# 一个json切分成两个struct
# 临时改名struct的字段
# 用字符串传递数字
# 容忍字符串和数字互转
# 容忍空数组作为对象
# 使用 MarshalJSON支持time.Time
# 使用 RegisterTypeEncoder支持time.Time
# 使用 MarshalText支持非字符串作为key的map
# 使用 json.RawMessage
# 使用 json.Number
# 统一更改字段的命名风格
# 使用私有的字段
