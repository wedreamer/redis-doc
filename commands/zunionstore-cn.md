计算`numkeys`由指定键给出的排序集, 以及
将结果存储在`destination`.
必须提供输入键的数量 (`numkeys`)  通过之前
输入键和其他 (可) ) 参数。

默认情况下, 元素的结果分数是其在
已排序的集合, 它位于其存在的位置。

使用`WEIGHTS`选项, 可以指定乘法因子
对于每个输入排序集。
这意味着每个输入排序集中每个元素的分数为
乘以此因子, 然后传递到聚合函数。
什么时候`WEIGHTS`未给出, 乘法因子默认为`1`.

随着`AGGREGATE`选项, 可以指定结果如何
并集是聚合的。
此选项默认为`SUM`, 其中元素的分数相加
它存在的输入。
当此选项设置为`MIN`或`MAX`, 结果集将包含
元素在存在元素的输入中的最小或最大分数。

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
ZUNIONSTORE out 2 zset1 zset2 WEIGHTS 2 3
ZRANGE out 0 -1 WITHSCORES
```
