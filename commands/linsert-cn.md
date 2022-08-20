插入`element`存储在 列表中`key`在引用之前或之后
价值`pivot`.

什么时候`key`不存在，它被视为空列表，并且没有操作是
执行。

在以下情况下返回错误`key`存在但不保存列表值。

@return

@integer回复：插入操作后列表的长度，或`-1`什么时候
值`pivot`未找到。

@examples

```cli
RPUSH mylist "Hello"
RPUSH mylist "World"
LINSERT mylist BEFORE "World" "There"
LRANGE mylist 0 -1
```
