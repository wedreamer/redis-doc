返回流中的条目数。如果指定的键没有
exist 命令返回零，就好像流是空的一样。
但请注意，与其他 Redis 类型不同，零长度流是
可能，所以你应该打电话`TYPE`或`EXISTS`为了检查是否
密钥是否存在。

一旦流内部没有条目，就不会自动删除（例如
在`XDEL`call），因为流可能有消费组
与之相关。

@return

@integer回复：流的条目数`key`.

@examples

```cli
XADD mystream * item 1
XADD mystream * item 2
XADD mystream * item 3
XLEN mystream
```
