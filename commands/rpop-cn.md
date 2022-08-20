删除并返回存储在`key`.

默认情况下，该命令从列表末尾弹出单个元素。
当提供可选`count`参数，回复将包括 up
自`count`元素，具体取决于列表的长度。

@return

在没有`count`论点：

@bulk字符串回复：最后一个元素的值，或`nil`什么时候`key`不存在。

当调用`count`论点：

@array回复：弹出元素列表，或`nil`什么时候`key`不存在。

@examples

```cli
RPUSH mylist "one" "two" "three" "four" "five"
RPOP mylist
RPOP mylist 2
LRANGE mylist 0 -1
```
