`BZPOPMIN`是排序集的阻塞变体`ZPOPMIN`原始。

它是阻止版本, 因为它在没有连接时阻止连接
从任何给定的排序集中弹出的成员。
得分最低的成员将从第一个排序集中弹出, 该集合是
非空, 按给定键的顺序检查给定的键。

这`timeout`参数被解释为指定最大值的双精度值
要阻止的秒数。超时值为零可用于无限期阻止。

查看[BLPOP 文档][cl]对于确切的语义, 因为`BZPOPMIN`是
与`BLPOP`唯一的区别是数据结构是
弹出。

[cl]: /commands/blpop

@return

@array回复：具体而言：

*   一个`nil`当无法弹出任何元素并且超时过期时, 多批量。
*   三元素多体积, 第一个元素是键的名称
    在弹出成员的地方, 第二个元素是弹出的成员本身, 
    第三个元素是弹出元素的分数。

@examples

    redis> DEL zset1 zset2
    (integer) 0
    redis> ZADD zset1 0 a 1 b 2 c
    (integer) 3
    redis> BZPOPMIN zset1 zset2 0
    1) "zset1"
    2) "a"
    3) "0"
