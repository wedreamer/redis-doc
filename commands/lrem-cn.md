删除第一个`count`元素的出现次数等于`element`从列表中
存储在`key`.
这`count`参数通过以下方式影响操作：

*   `count > 0`：删除等于`element`从头到尾移动。
*   `count < 0`：删除等于`element`从尾巴移动到头部。
*   `count = 0`：删除等于`element`.

例如`LREM list -2 "hello"`将删除最后两次出现的
`"hello"`存储在 列表中`list`.

请注意，不存在的键被视为空列表，因此当`key`不
存在，命令将始终返回`0`.

@return

@integer回复：已删除元素的数量。

@examples

```cli
RPUSH mylist "hello"
RPUSH mylist "hello"
RPUSH mylist "foo"
RPUSH mylist "hello"
LREM mylist -2 "hello"
LRANGE mylist 0 -1
```
