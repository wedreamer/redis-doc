返回存储在`key`.
如果`key`不存在, 它被解释为一个空列表和`0`返回。
当值存储在`key`不是列表。

@return

@integer回复：列表的长度`key`.

@examples

```cli
LPUSH mylist "World"
LPUSH mylist "Hello"
LLEN mylist
```
