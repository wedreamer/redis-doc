返回排序集中的所有元素, 地址为`key`分数介于`max`
和`min` (包括分数等于`max`或`min`).
与排序集的默认顺序相反, 对于此命令, 
元素被认为是从高分到低分的顺序。

具有相同分数的元素以反向词典形式返回
次序。

除了颠倒的顺序, `ZREVRANGEBYSCORE`类似于
`ZRANGEBYSCORE`.

@return

@array回复：指定分数范围内的元素列表 (可选) 
与他们的分数) 。

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZADD myzset 3 "three"
ZREVRANGEBYSCORE myzset +inf -inf
ZREVRANGEBYSCORE myzset 2 1
ZREVRANGEBYSCORE myzset 2 (1
ZREVRANGEBYSCORE myzset (2 (1
```
