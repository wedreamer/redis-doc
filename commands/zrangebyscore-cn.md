返回排序集中的所有元素，地址为`key`分数介于`min`
和`max`（包括分数等于`min`或`max`).
元素被认为是从低分到高分的顺序。

具有相同分数的元素按词典顺序返回（this
遵循 Redis 中排序集实现的属性，并且不
涉及进一步的计算）。

可选`LIMIT`参数可用于仅获取匹配范围的参数
元素（类似于*选择极限偏移，计数*在 SQL 中）。否定`count`
返回`offset`.
请记住，如果`offset`较大，需要遍历的排序集
`offset`元素之前要返回的元素，这可以加起来
O（N） 时间复杂度。

可选`WITHSCORES`参数使命令同时返回元素和
它的分数，而不是单独的元素。
此选项自 Redis 2.0 起可用。

## 独占间隔和无穷大

`min`和`max`可以是`-inf`和`+inf`，这样您就不需要知道
排序集中的最高或最低分数，用于获取来自或达到
一定的分数。

默认情况下，指定的间隔为`min`和`max`已关闭（包括）。
可以通过在分数前面加上前缀来指定开放间隔（独占）
与角色`(`.
例如：

    ZRANGEBYSCORE zset (1 5

将返回所有元素`1 < score <= 5`而：

    ZRANGEBYSCORE zset (5 (10

将返回所有元素`5 < score < 10`（不包括 5 和 10）。

@return

@array回复：指定分数范围内的元素列表（可选）
与他们的分数）。

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZADD myzset 3 "three"
ZRANGEBYSCORE myzset -inf +inf
ZRANGEBYSCORE myzset 1 2
ZRANGEBYSCORE myzset (1 2
ZRANGEBYSCORE myzset (1 (2
```

## 模式：元素的加权随机选择

通常`ZRANGEBYSCORE`仅用于获取项目范围
其中分数是索引整数键，但可以做得更少
命令明显的东西。

例如，实现马尔可夫链和其他算法时的常见问题
是从集合中随机选择一个元素，但不同的元素可能具有
不同的权重，改变他们被挑选的可能性。

这就是我们如何使用此命令来挂载这样的算法：

想象一下，你有权重为1，2和3的元素A，B和C。
计算权重之和，即 1+2+3 = 6

此时，您可以使用以下算法将所有元素添加到排序集中：

    SUM = ELEMENTS.TOTAL_WEIGHT // 6 in this case.
    SCORE = 0
    FOREACH ELE in ELEMENTS
        SCORE += ELE.weight / SUM
        ZADD KEY SCORE ELE
    END

这意味着您设置：

    A to score 0.16
    B to score .5
    C to score 1

由于这涉及近似，为了避免C被设置为，
比如，0.998而不是1，我们只是修改上面的算法以确保
最后一个分数是1（留给读者练习...）。

此时，每次要获取加权随机元素时，
只需计算一个介于 0 和 1 之间的随机数（这就像调用
`rand()`在大多数语言中），所以你可以只做：

    RANDOM_ELE = ZRANGEBYSCORE key RAND() +inf LIMIT 0 1
