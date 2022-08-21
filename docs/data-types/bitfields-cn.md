---
title: "Redis bitfields"
linkTitle: "Bitfields"
weight: 130
description: >
    Introduction to Redis bitfields
---


Redis 位字段允许您设置、递增和获取任意位长度的整数值。
例如，您可以对从无符号 1 位整数到有符号 63 位整数的任何内容进行操作。

这些值使用二进制编码的 Redis 字符串进行存储。
Bitfields 支持原子读取，写入和增量操作，使其成为管理计数器和类似数值的不错选择。

## 例子

假设您正在跟踪在线游戏中的活动。
你要为每个玩家保留两个关键指标：黄金总量和杀死的怪物数量。
由于您的游戏非常容易上瘾，因此这些计数器的宽度应至少为 32 位。

您可以使用每个玩家一个位字段来表示这些计数器。

*   新玩家以1000金币开始教程（计数器偏移量为0）。

<!---->

    > BITFIELD player:1:stats SET u32 #0 1000
    1) (integer) 0

*   杀死俘虏王子的妖精后，添加赚取的50金币并增加 "被杀" 计数器（偏移1）。

<!---->

    > BITFIELD player:1:stats INCRBY u32 #0 50 INCRBY u32 #1 1
    1) (integer) 1050
    2) (integer) 1

*   付给铁匠999金币买一把传说中的生锈匕首。

<!---->

    > BITFIELD player:1:stats INCRBY u32 #0 -999
    1) (integer) 51

*   阅读玩家的统计数据：

<!---->

    > BITFIELD player:1:stats GET u32 #0 GET u32 #1
    1) (integer) 51
    2) (integer) 1

## 基本命令

*   `BITFIELD`原子地设置、递增和读取一个或多个值。
*   `BITFIELD_RO`是 的只读变体`BITFIELD`.

## 性能

`BITFIELD`是 O（n），其中 *n* 是访问的计数器数。
