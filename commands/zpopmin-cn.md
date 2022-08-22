删除并返回`count`排序中得分最低的成员
集存储在`key`.

如果未指定, 则`count`为 1。指定`count`
高于排序集的基数的值将不会生成
错误。返回多个元素时, 得分最低的元素将
是第一个, 其次是得分更高的元素。

@return

@array回复：弹出的元素和分数列表。

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZADD myzset 3 "three"
ZPOPMIN myzset
```
