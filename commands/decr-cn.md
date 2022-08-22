递减存储在`key`由一个。
如果该项不存在, 则将其设置为`0`在执行操作之前。
如果键包含错误类型的值或包含
不能表示为整数的字符串。
此操作仅限于**64 位有符号整数**.

看`INCR`以获取有关递增/递减操作的额外信息。

@return

@integer回复：值`key`递减后

@examples

```cli
SET mykey "10"
DECR mykey
SET mykey "234293482390480948029348230948"
DECR mykey
```
