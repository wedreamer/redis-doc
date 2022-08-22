返回存储在`key`.
偏移量`start`和`stop`是从零开始的索引, 具有`0`成为第一个
元素 (列表的头部) , `1`作为下一个元素, 等等
上。

这些偏移量也可以是负数, 表示从
列表的末尾。
例如`-1`是列表的最后一个元素, `-2`倒数第二个, 等等
上。

## 与各种编程语言中的范围函数保持一致

请注意, 如果您有一个从 0 到 100 的数字列表, `LRANGE list 0 10`将
返回 11 个元素, 即包含最右边的项。
这**可能也可能不可以**与范围相关功能的行为一致
在你选择的编程语言中 (想想Ruby的`Range.new`,`Array#slice`
或 Python 的`range()`函数) 。

## 超出范围的索引

超出范围的索引不会产生错误。
如果`start`大于列表的末尾, 则返回空列表。
如果`stop`大于列表的实际末尾, Redis 会像对待它一样对待它
列表的最后一个元素。

@return

@array回复：指定范围内的元素列表。

@examples

```cli
RPUSH mylist "one"
RPUSH mylist "two"
RPUSH mylist "three"
LRANGE mylist 0 0
LRANGE mylist -3 2
LRANGE mylist -100 100
LRANGE mylist 5 10
```
