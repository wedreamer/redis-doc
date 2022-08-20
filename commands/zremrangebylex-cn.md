当以相同的分数插入已排序集中的所有元素时，为了强制进行词典排序，此命令将删除存储在`key`在指定的词典范围之间`min`和`max`.

的含义`min`和`max`是相同的`ZRANGEBYLEX`命令。同样，此命令实际上删除了与`ZRANGEBYLEX`如果以相同的方式调用，将返回`min`和`max`参数。

@return

@integer回复：删除的元素数。

@examples

```cli
ZADD myzset 0 aaaa 0 b 0 c 0 d 0 e
ZADD myzset 0 foo 0 zap 0 zip 0 ALPHA 0 alpha
ZRANGE myzset 0 -1
ZREMRANGEBYLEX myzset [alpha [omega
ZRANGE myzset 0 -1
```
