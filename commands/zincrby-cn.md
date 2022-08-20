将分数递增`member`在排序集中存储在`key`由
`increment`.
如果`member`在排序集中不存在，它被添加`increment`如
它的分数（好像它以前的分数是`0.0`).
如果`key`不存在，具有指定值的新排序集`member`作为其
创建唯一成员。

在以下情况下返回错误`key`存在但不保存已排序的集合。

这`score`value 应该是数值的字符串表示形式，并且
接受双精度浮点数。
可以提供负值来递减分数。

@return

@bulk字符串回复：新乐谱`member`（双精度浮点
数字），表示为字符串。

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZINCRBY myzset 2 "one"
ZRANGE myzset 0 -1 WITHSCORES
```
