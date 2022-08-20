返回排序集中的元素数，位于`key`分数介于
`min`和`max`.

这`min`和`max`参数具有与 所描述的相同的语义
`ZRANGEBYSCORE`.

注意：该命令的复杂性仅为 O（log（N）），因为它使用元素等级（请参阅`ZRANK`） 来了解范围。因此，没有必要做与范围大小成比例的工作。

@return

@integer回复：指定分数范围内的元素数。

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZADD myzset 3 "three"
ZCOUNT myzset -inf +inf
ZCOUNT myzset (1 3
```
