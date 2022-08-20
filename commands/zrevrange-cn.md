返回存储在`key`.
元素被视为从最高到最低分数的顺序排列。
降序词典顺序用于得分相等的元素。

除了颠倒的顺序，`ZREVRANGE`类似于`ZRANGE`.

@return

@array回复：指定范围内的元素列表（可选
他们的分数）。

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZADD myzset 3 "three"
ZREVRANGE myzset 0 -1
ZREVRANGE myzset 2 3
ZREVRANGE myzset -2 -1
```
