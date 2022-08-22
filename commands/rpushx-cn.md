在存储在`key`, 仅当`key`
已存在并持有一个列表。
与`RPUSH`, 则在以下情况下不会执行任何操作`key`尚未
存在。

@return

@integer回复：推送操作后列表的长度。

@examples

```cli
RPUSH mylist "Hello"
RPUSHX mylist "World"
RPUSHX myotherlist "World"
LRANGE mylist 0 -1
LRANGE myotherlist 0 -1
```
