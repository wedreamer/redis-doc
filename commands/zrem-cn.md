从存储在 的排序集中删除指定成员`key`.
将忽略不存在的成员。

在以下情况下返回错误`key`存在并且不保存已排序的集合。

@return

@integer回复, 具体而言：

*   从已排序集中删除的成员数, 不包括不存在的成员数
    成员。

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZADD myzset 3 "three"
ZREM myzset "two"
ZRANGE myzset 0 -1 WITHSCORES
```
