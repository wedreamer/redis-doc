---
title: "Redis HyperLogLog"
linkTitle: "HyperLogLog"
weight: 90
description: >
    Introduction to the Redis HyperLogLog data type
---

HyperLogLog 是一种数据结构, 用于估计集合的基数。作为概率数据结构, HyperLogLog 以完美的精度换取高效的空间利用率。

Redis HyperLogLog 实现最多使用 12 KB, 并提供 0.81% 的标准误差。

## 例子

*   向 HyperLogLog 中添加一些项目：

<!---->

    > PFADD members 123
    (integer) 1
    > PFADD members 500
    (integer) 1
    > PFADD members 12
    (integer) 1

*   估计集合中的成员数：

<!---->

    > PFCOUNT members
    (integer) 3

## 基本命令

*   `PFADD`将项目添加到 HyperLogLog。
*   `PFCOUNT`返回集合中项数的估计值。
*   `PFMERGE`将两个或多个 HyperLogs 合并为一个。

查看[HyperLogLog 命令的完整列表](https://redis.io/commands/?group=hyperloglog).

## 性能

写作 (`PFADD`)  到 并读取  (`PFCOUNT`) 的HyperLogLog是在恒定的时间和空间中完成的。
合并 HLL 为 O (n) , 其中 *n* 是草图的数量。

## 限制

HyperLogLog 可以估计最多具有 18, 446, 744, 073, 709, 551, 616  (2^64)  个成员的集合的基数。

## 了解更多信息

*   [Redis 新的数据结构：HyperLogLog](https://antirez.com/news/75)有很多关于数据结构及其在Redis中的实现的细节。
*   [Redis HyperLogLog Explained](https://www.youtube.com/watch?v=MunL8nnwscQ)向您展示如何使用 Redis HyperLogLog 数据结构来构建流量热图。
