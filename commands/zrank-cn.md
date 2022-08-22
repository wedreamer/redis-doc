返回`member`在排序集中存储在`key`, 以及分数
从低到高排序。
排名 (或索引) 从 0 , 始, 这意味着成员的最低值
分数有排名`0`.

用`ZREVRANK`获取分数从高排序的元素的排名
到低。

@return

*   如果`member`存在于排序集中, @integer回复：的秩`member`.
*   如果`member`不存在于已排序的集中, 或者`key`不存在, 
    @bulk字符串回复：`nil`.

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZADD myzset 3 "three"
ZRANK myzset "three"
ZRANK myzset "four"
```
