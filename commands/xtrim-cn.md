`XTRIM`如果需要, 通过逐出较旧的条目 (ID 较低的条目) 来修剪流。

可以使用以下策略之一完成修剪流：

*   `MAXLEN`：只要流的长度超过指定的长度, 就会逐出条目`threshold`哪里`threshold`是正整数。
*   `MINID`：逐出 ID 低于以下的条目`threshold`哪里`threshold`是流 ID。

例如, 这会将流修剪为恰好最新的 1000 个项目：

    XTRIM mystream MAXLEN 1000

而在此示例中, ID 低于 649085820-0 的所有条目都将被逐出：

    XTRIM mystream MINID 649085820

默认情况下, 或当提供可选`=`参数, 则该命令执行精确修剪。

根据策略的不同, 精确修剪意味着：

*   `MAXLEN`：修剪后的流的长度将正好是其原始长度与指定长度之间的最小值`threshold`.
*   `MINID`：流中最早的 ID 将正好是其原始最旧 ID 与指定 ID 之间的最大值`threshold`.

## 近乎精确的修剪

由于精确的修剪可能需要 Redis 服务器进行额外的工作, 因此可选`~`可以提供参数以使其更有效。

例如：

    XTRIM mystream MAXLEN ~ 1000

这`~`参数之间的参数`MAXLEN`策略和`threshold`表示用户请求修剪流, 使其长度为**至少**这`threshold`, 但可能略多一些。
在这种情况下, 当可以获得性能时 (例, , 当无法删除数据结构中的整个宏节点), ) , Redis 将提前停止修剪。
这使得修剪更加高效, 并且通常是您想要的, 尽管在修剪之后, 流可能只有几十个额外的条目`threshold`.

控制命令在使用 时完成的工作量的另一种方法`~`, 是`LIMIT`第。
使用时, 它指定最大值`count`将被逐出的条目。
什么时候`LIMIT`和`count`未指定, 默认值 100 \* 宏节点中的条目数将隐式用作`count`.
将值 0 指定为`count`完全禁用限制机制。

@return

@integer回复：从流中删除的条目数。

@examples

```cli
XADD mystream * field1 A field2 B field3 C field4 D
XTRIM mystream MAXLEN 2
XRANGE mystream - +
```
