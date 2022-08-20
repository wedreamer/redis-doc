这`LATENCY HISTOGRAM`命令以直方图格式报告每个指定命令名称的延迟累积分布。
如果未指定命令名称，则将回复包含延迟信息的所有命令。

每个报告的直方图都具有以下字段：

*   命令名称。
*   对该命令的总调用。
*   时间桶地图：
    *   每个存储桶代表一个延迟范围。
    *   每个存储桶覆盖前一个存储桶范围的两倍。
    *   不打印空存储桶。
    *   跟踪的延迟介于 1 微秒和大约 1 秒之间。
    *   超过 1 秒的所有内容都被视为 +Inf。
    *   最多将有 log2（1000000000）=30 个存储桶。

此命令需要启用延长延迟监视功能（默认情况下已启用）。
如果需要启用它，请使用`CONFIG SET latency-tracking yes`.

@examples

    127.0.0.1:6379> LATENCY HISTOGRAM set
    1# "set" =>
       1# "calls" => (integer) 100000
       2# "histogram_usec" =>
          1# (integer) 1 => (integer) 99583
          2# (integer) 2 => (integer) 99852
          3# (integer) 4 => (integer) 99914
          4# (integer) 8 => (integer) 99940
          5# (integer) 16 => (integer) 99968
          6# (integer) 33 => (integer) 100000

@return

@array回复：具体而言：

该命令返回一个映射，其中每个键是一个命令名称，每个值是一个包含总调用次数的映射，以及直方图时间存储桶的内部映射。
