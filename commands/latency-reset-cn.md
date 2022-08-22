这`LATENCY RESET`命令重置所有或仅部分事件的延迟峰值时间序列。

当调用不带参数的命令时, 它将重置所有
事件, 丢弃当前记录的延迟峰值事件, 并重置
最大事件时间寄存器。

可以通过提供`event`名字
作为参数。

的有效值`event`是：

*   `active-defrag-cycle`
*   `aof-fsync-always`
*   `aof-stat`
*   `aof-rewrite-diff-write`
*   `aof-rename`
*   `aof-write`
*   `aof-write-active-child`
*   `aof-write-alone`
*   `aof-write-pending-fsync`
*   `command`
*   `expire-cycle`
*   `eviction-cycle`
*   `eviction-del`
*   `fast-command`
*   `fork`
*   `rdb-unlink-temp-file`

有关详细信息, 请参阅[延迟监控框架页面][lm].

[lm]: /topics/latency-monitor

@return

@integer回复：重置的事件时间序列的数量。
