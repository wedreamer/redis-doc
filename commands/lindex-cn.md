返回索引处的元素`index`存储在 列表中`key`.
索引是从零开始的，因此`0`表示第一个元素，`1`第二个元素
等等。
负指数可用于指定从尾部开始的元素
列表。
这里`-1`表示最后一个元素，`-2`表示倒数第二个，依此类推。

当值在`key`不是列表，则返回错误。

@return

@bulk字符串回复：请求的元素，或`nil`什么时候`index`超出范围。

@examples

```cli
LPUSH mylist "World"
LPUSH mylist "Hello"
LINDEX mylist 0
LINDEX mylist -1
LINDEX mylist 3
```
