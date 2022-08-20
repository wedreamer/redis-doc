将所有元素参数添加到以指定为第一个参数的变量名称存储的 HyperLogLog 数据结构中。

作为此命令的副作用，HyperLogLog 内部可能会更新，以反映对到目前为止添加的唯一项目数（集合的基数）的不同估计。

如果 HyperLogLog 估计的近似基数在执行命令后发生更改，`PFADD`返回 1，否则返回 0。如果指定的键不存在，则该命令会自动创建一个空的 HyperLogLog 结构（即，具有指定长度且具有给定编码的 Redis 字符串）。

要调用不带元素但只有变量名称有效的命令，如果变量已存在，这将导致不执行任何操作，或者如果键不存在，则仅创建数据结构（在后一种情况下返回 1）。

有关 HyperLogLog 数据结构的介绍，请查看`PFCOUNT`命令页。

@return

@integer回复，具体而言：

*   如果至少更改了 1 个 HyperLogLog 内部寄存器，则为 1。否则为 0。

@examples

```cli
PFADD hll a b c d e f g
PFCOUNT hll
```
