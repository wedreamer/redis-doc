删除排序集中存储在 的所有元素`key`分数介于
`min`和`max` (包括) 。

@return

@integer回复：删除的元素数。

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZADD myzset 3 "three"
ZREMRANGEBYSCORE myzset -inf (2
ZRANGE myzset 0 -1 WITHSCORES
```
