---
title: "Redis Streams"
linkTitle: "Streams"
weight: 60
description: >
    Introduction to Redis streams
---

Redis 流是一种数据结构, 其作用类似于仅追加日志。
您可以使用流实时记录和同时联合事件。
Redis 流用例的示例包括：

*   事件溯源 (例如, 跟踪用户操作、点击次数等) 
*   传感器监控 (例如, 来自现场设备的读数) 
*   通知 (例如, 将每个用户的通知记录存储在单独的流中) 

Redis 为每个流条目生成一个唯一的 ID。
您可以使用这些 ID 稍后检索其关联的条目, 或者读取和处理流中的所有后续条目。

Redis 流支持多种修剪策略 (以防止流无限增长) 和多个使用策略 (请参阅`XREAD`,`XREADGROUP`和`XRANGE`).

## 例子

*   向流中添加多个温度读数

<!---->

    > XADD temperatures:us-ny:10007 * temp_f 87.2 pressure 29.69 humidity 46
    "1658354918398-0"
    > XADD temperatures:us-ny:10007 * temp_f 83.1 pressure 29.21 humidity 46.5
    "1658354934941-0"
    > XADD temperatures:us-ny:10007 * temp_f 81.9 pressure 28.37 humidity 43.7
    "1658354957524-0"

*   读取从 ID 开始的前两个流条目`1658354934941-0`:

<!---->

    > XRANGE temperatures:us-ny:10007 1658354934941-0 + COUNT 2
    1) 1) "1658354934941-0"
       2) 1) "temp_f"
          2) "83.1"
          3) "pressure"
          4) "29.21"
          5) "humidity"
          6) "46.5"
    2) 1) "1658354957524-0"
       2) 1) "temp_f"
          2) "81.9"
          3) "pressure"
          4) "28.37"
          5) "humidity"
          6) "43.7"

*   从流结束时开始, 读取最多 100 个新流条目, 如果未写入任何条目, 则阻止最多 300 毫秒：

<!---->

    > XREAD COUNT 100 BLOCK 300 STREAMS tempertures:us-ny:10007 $
    (nil)

## 基本命令

*   `XADD`将新条目添加到流中。
*   `XREAD`读取一个或多个条目, 从给定位置开始, 并及时向前移动。
*   `XRANGE`返回两个提供的条目 ID 之间的条目范围。
*   `XLEN`返回流的长度。

查看[流命令的完整列表](https://redis.io/commands/?group=stream).

## 性能

向流中添加条目是 O (1) 。
访问任何单个条目是 O (n) , 其中*n*是 ID 的长度。
由于流 ID 通常较短且长度固定, 因此这有效地减少到恒定时间查找。
有关原因的详细信息, 请注意, 流实现为[基数树](https://en.wikipedia.org/wiki/Radix_tree).

简而言之, Redis 流提供高效的插入和读取。
有关详细信息, 请参阅每个命令的时间复杂度。

## 了解更多信息

*   这[Redis Streams Tutorial](/docs/data-types/streams-tutorial)用许多例子解释了Redis流。
*   [Redis Streams Explained](https://www.youtube.com/watch?v=Z8qcpXyMAiA)是对Redis中流的有趣介绍。
*   [Redis University's RU102](https://university.redis.com/courses/ru102/)是一个专门针对 Redis 流的免费在线课程。
