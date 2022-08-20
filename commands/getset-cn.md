原子集合`key`自`value`并返回存储在`key`.
在以下情况下返回错误`key`存在但不保存字符串值。 任何
与密钥关联的上一个生存时间在成功时被丢弃
`SET`操作。

## 设计模式

`GETSET`可与`INCR`用于原子复位计数。
例如：一个进程可能调用`INCR`针对密钥`mycounter`每次
一些事件发生，但不时我们需要得到计数器的值
并以原子方式将其重置为零。
这可以通过以下方式完成`GETSET mycounter "0"`:

```cli
INCR mycounter
GETSET mycounter "0"
GET mycounter
```

@return

@bulk字符串回复：存储在`key`或`nil`什么时候`key`不存在。

@examples

```cli
SET mykey "Hello"
GETSET mykey "World"
GET mykey
```
