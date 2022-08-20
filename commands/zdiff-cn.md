此命令类似于`ZDIFFSTORE`，而不是存储结果
排序集，它将返回到客户端。

@return

@array回复：差异的结果（如果
这`WITHSCORES`给出了选项）。

@examples

```cli
ZADD zset1 1 "one"
ZADD zset1 2 "two"
ZADD zset1 3 "three"
ZADD zset2 1 "one"
ZADD zset2 2 "two"
ZDIFF 2 zset1 zset2
ZDIFF 2 zset1 zset2 WITHSCORES
```
