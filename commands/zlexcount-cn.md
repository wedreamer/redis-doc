当以相同的分数插入已排序集中的所有元素时，为了强制进行词典排序，此命令返回排序集中的元素数，位于`key`具有介于`min`和`max`.

这`min`和`max`参数与 描述的含义相同
`ZRANGEBYLEX`.

注意：该命令的复杂性仅为 O（log（N）），因为它使用元素等级（请参阅`ZRANK`） 来了解范围。因此，没有必要做与范围大小成比例的工作。

@return

@integer回复：指定分数范围内的元素数。

@examples

```cli
ZADD myzset 0 a 0 b 0 c 0 d 0 e
ZADD myzset 0 f 0 g
ZLEXCOUNT myzset - +
ZLEXCOUNT myzset [b [f
```
