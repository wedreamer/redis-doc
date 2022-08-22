计算`numkeys`由指定键给出的排序集, 
并将结果存储在`destination`.
必须提供输入键的数量 (`numkeys`)  通过之前
输入键和其他 (可) ) 参数。

默认情况下, 元素的结果分数是其在
已排序的集合, 它位于其存在的位置。
因为交集要求一个元素是每个给定排序的成员
set, 这将导致生成的排序集中每个元素的分数为
等于输入排序集的数量。

有关`WEIGHTS`和`AGGREGATE`选项, 请参阅`ZUNIONSTORE`.

如果`destination`已经存在, 它被覆盖了。

@return

@integer回复：生成的排序集中的元素数, 位于
`destination`.

@examples

```cli
ZADD zset1 1 "one"
ZADD zset1 2 "two"
ZADD zset2 1 "one"
ZADD zset2 2 "two"
ZADD zset2 3 "three"
ZINTERSTORE out 2 zset1 zset2 WEIGHTS 2 3
ZRANGE out 0 -1 WITHSCORES
```
