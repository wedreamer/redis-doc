移动`member`从集合在`source`到集合`destination`.
此操作是原子操作。
在每个给定的时刻, 该元素将显示为`source` **或**
`destination`对于其他客户端。

如果源集不存在或不包含指定的元素, 则否
执行操作, 然后`0`返回。
否则, 该元素将从源代码集中删除并添加到
目标集。
当目标集中已存在指定的元素时, 它仅
从源集中删除。

在以下情况下返回错误：`source`或`destination`不保存设定值。

@return

@integer回复, 具体而言：

*   `1`如果元素被移动。
*   `0`如果元素不是 的成员`source`并且未执行任何操作。

@examples

```cli
SADD myset "one"
SADD myset "two"
SADD myotherset "three"
SMOVE myset myotherset "two"
SMEMBERS myset
SMEMBERS myotherset
```
