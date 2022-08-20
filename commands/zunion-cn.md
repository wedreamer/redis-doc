此命令类似于`ZUNIONSTORE`，而不是存储结果
排序集，它将返回到客户端。

有关`WEIGHTS`和`AGGREGATE`选项，请参阅`ZUNIONSTORE`.

@return

@array回复：并集的结果（如果
这`WITHSCORES`给出了选项）。

@examples

```cli
ZADD zset1 1 "one"
ZADD zset1 2 "two"
ZADD zset2 1 "one"
ZADD zset2 2 "two"
ZADD zset2 3 "three"
ZUNION 2 zset1 zset2
ZUNION 2 zset1 zset2 WITHSCORES
```
