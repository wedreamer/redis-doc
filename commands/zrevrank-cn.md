返回`member`在排序集中存储在`key`，以及分数
从高到低排序。
排名（或索引）从 0 开始，这意味着具有最高值的成员
分数有排名`0`.

用`ZRANK`获取分数从低到
高。

@return

*   如果`member`存在于排序集中，@integer回复：的秩`member`.
*   如果`member`不存在于已排序的集中，或者`key`不存在，
    @bulk字符串回复：`nil`.

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZADD myzset 3 "three"
ZREVRANK myzset "one"
ZREVRANK myzset "four"
```
