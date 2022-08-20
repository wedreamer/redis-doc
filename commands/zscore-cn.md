返回`member`在排序集的`key`.

如果`member`在排序集中不存在，或者`key`不存在，`nil`是
返回。

@return

@bulk字符串回复：分数`member`（双精度浮点数），
表示为字符串。

@examples

```cli
ZADD myzset 1 "one"
ZSCORE myzset "one"
```
