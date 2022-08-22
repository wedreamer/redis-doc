将列表元素设置为`index`自`element`.
有关`index`参数, 请参阅`LINDEX`.

对于超出范围的索引, 将返回错误。

@return

@simple字符串回复

@examples

```cli
RPUSH mylist "one"
RPUSH mylist "two"
RPUSH mylist "three"
LSET mylist 0 "four"
LSET mylist -2 "five"
LRANGE mylist 0 -1
```
