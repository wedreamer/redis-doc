删除排序集中存储在 的所有元素`key`等级介于`start`
和`stop`.
双`start`和`stop`是`0`基于索引`0`成为
最低分数。
这些索引可以是负数, 其中它们指示从
得分最高的元素。
例如：`-1`是得分最高的元素, `-2`元素
第二高分, 依此类推。

@return

@integer回复：删除的元素数。

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZADD myzset 3 "three"
ZREMRANGEBYRANK myzset 0 1
ZRANGE myzset 0 -1 WITHSCORES
```
