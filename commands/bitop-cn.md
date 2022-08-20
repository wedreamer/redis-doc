在多个键（包含字符串值）和
将结果存储在目标密钥中。

这`BITOP`命令支持四种按位操作：**和**,**或**,**异或**
和**不**，因此调用命令的有效形式为：

*   `BITOP AND destkey srckey1 srckey2 srckey3 ... srckeyN`
*   `BITOP OR  destkey srckey1 srckey2 srckey3 ... srckeyN`
*   `BITOP XOR destkey srckey1 srckey2 srckey3 ... srckeyN`
*   `BITOP NOT destkey srckey`

如您所见**不**很特别，因为它只需要一个输入键，因为它
执行位的反演，因此它仅作为一元运算符才有意义。

操作结果始终存储在`destkey`.

## 处理不同长度的字符串

当在具有不同长度的字符串之间执行操作时，所有
短于集合中最长字符串的字符串将被视为
零填充，直到最长字符串的长度。

这同样适用于不存在的密钥，这些密钥被视为
最长字符串长度为零字节。

@return

@integer回复

存储在目标键中的字符串的大小，等于
最长输入字符串的大小。

@examples

```cli
SET key1 "foobar"
SET key2 "abcdef"
BITOP AND dest key1 key2
GET dest
```

## 模式：使用位图的实时指标

`BITOP`是对`BITCOUNT`命令
文档。
可以组合不同的位图，以获得目标位图，其中
执行人口计数操作。

请参阅名为”[使用 Redis 快速、轻松的实时指标
位图][hbgc212fermurb]“为一个有趣的用例。

[hbgc212fermurb]: http://blog.getspool.com/2011/11/29/fast-easy-realtime-metrics-using-redis-bitmaps

## 性能注意事项

`BITOP`是一个可能很慢的命令，因为它在 O（N） 时间内运行。
对长输入字符串运行它时应小心。

对于涉及大量输入的实时指标和统计数据，一个好的方法是
使用副本（禁用只读选项），其中按位
执行操作以避免阻塞主实例。
