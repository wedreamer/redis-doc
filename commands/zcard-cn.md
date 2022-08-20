返回存储的已排序集的排序集基数（元素数）
在`key`.

@return

@integer回复：排序集的基数（元素数），或`0`
如果`key`不存在。

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZCARD myzset
```
