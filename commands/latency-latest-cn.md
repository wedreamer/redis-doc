这`LATENCY LATEST`命令报告记录的最新延迟事件。

每个报告的事件都有以下字段：

*   事件名称。
*   事件的最新延迟峰值的 Unix 时间戳。
*   最新事件延迟（以毫秒为单位）。
*   此事件的所有时间最大延迟。

“所有时间”表示自 Redis 实例以来的最大延迟
已开始，或事件重置的时间`LATENCY RESET`.

@examples

    127.0.0.1:6379> debug sleep 1
    OK
    (1.00s)
    127.0.0.1:6379> debug sleep .25
    OK
    127.0.0.1:6379> latency latest
    1) 1) "command"
       2) (integer) 1405067976
       3) (integer) 251
       4) (integer) 1001

有关详细信息，请参阅[延迟监控框架页面][lm].

[lm]: /topics/latency-monitor

@return

@array回复：具体而言：

该命令返回一个数组，其中每个元素都是一个四元素数组
表示事件的名称、时间戳、最新和所有时间延迟测量值。
