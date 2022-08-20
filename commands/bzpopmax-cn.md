`BZPOPMAX`是排序集的阻塞变体`ZPOPMAX`原始。

它是阻止版本，因为它在没有连接时阻止连接
从任何给定的排序集中弹出的成员。
得分最高的成员将从第一个排序集中弹出，该集合是
非空，按给定键的顺序检查给定的键。

这`timeout`参数被解释为指定最大值的双精度值
要阻止的秒数。超时值为零可用于无限期阻止。

查看[BZPOPMIN 文档][cb]对于确切的语义，因为`BZPOPMAX`
与`BZPOPMIN`唯一的区别是它弹出成员
具有最高分数，而不是以最低分数弹出的分数。

[cb]: /commands/bzpopmin

@return

@array回复：具体而言：

*   一个`nil`当无法弹出任何元素并且超时过期时，多批量。
*   三元素多体积，第一个元素是键的名称
    在弹出成员的地方，第二个元素是弹出的成员本身，
    第三个元素是弹出元素的分数。

@examples

    redis> DEL zset1 zset2
    (integer) 0
    redis> ZADD zset1 0 a 1 b 2 c
    (integer) 3
    redis> BZPOPMAX zset1 zset2 0
    1) "zset1"
    2) "c"
    3) "2"
