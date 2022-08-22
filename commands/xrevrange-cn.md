此命令与`XRANGE`, 但与
以相反的顺序返回条目, 并采用开始-结束
范围反序：`XREVRANGE`您需要声明*结束*编号
以及后来的*开始*ID, 命令将生成所有元素
介于 (或完全类似) 两个 ID , 间, 从*结束*边。

例如, 获取从较高 ID 到较低 ID 的所有元素
可以使用的 ID：

    XREVRANGE somestream + -

类似地, 要只获取添加到流中的最后一个元素, 它是
足以发送：

    XREVRANGE somestream + - COUNT 1

@return

@array回复, 具体而言：

该命令返回 ID 与指定范围匹配的条目, 
从较高的 ID 到较低的 ID 匹配。
返回的条目已完成, 这意味着 ID 和所有字段
它们组成被返回。此外, 条目返回
它们的字段和值的顺序与`XADD`添加了它们。

@examples

```cli
XADD writers * name Virginia surname Woolf
XADD writers * name Jane surname Austen
XADD writers * name Toni surname Morrison
XADD writers * name Agatha surname Christie
XADD writers * name Ngozi surname Adichie
XLEN writers
XREVRANGE writers + - COUNT 1
```
