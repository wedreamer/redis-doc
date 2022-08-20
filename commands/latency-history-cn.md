这`LATENCY HISTORY`命令返回`event`的延迟峰值时间序列。

这对于想要获取原始数据以执行监视、显示图形等的应用程序非常有用。

该命令将为`event`.

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

@examples

    127.0.0.1:6379> latency history command
    1) 1) (integer) 1405067822
       2) (integer) 251
    2) 1) (integer) 1405067941
       2) (integer) 1001

有关详细信息，请参阅[延迟监控框架页面][lm].

[lm]: /topics/latency-monitor

@return

@array回复：具体而言：

该命令返回一个数组，其中每个元素都是一个双元素数组
表示事件的时间戳和延迟。
