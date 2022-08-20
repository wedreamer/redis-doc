当以相同的分数插入已排序集中的所有元素时，为了强制进行词典排序，此命令将返回排序集中的所有元素，地址为`key`具有介于`max`和`min`.

除了颠倒的顺序，`ZREVRANGEBYLEX`类似于`ZRANGEBYLEX`.

@return

@array回复：指定分数范围内的元素列表。

@examples

```cli
ZADD myzset 0 a 0 b 0 c 0 d 0 e 0 f 0 g
ZREVRANGEBYLEX myzset [c -
ZREVRANGEBYLEX myzset (c -
ZREVRANGEBYLEX myzset (g [aaa
```
