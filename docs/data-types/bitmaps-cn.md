---
title: "Redis bitmaps"
linkTitle: "Bitmaps"
weight: 120
description: >
    Introduction to Redis bitmaps
---

Redis 位图是字符串数据类型的扩展，允许您将字符串视为位向量。
还可以对一个或多个字符串执行按位运算。
位图用例的一些示例包括：

*   对于集合的成员对应于整数 0-N 的情况，有效的集合表示。
*   对象权限，其中每个位表示一个特定权限，类似于文件系统存储权限的方式。

## 例子

假设您在现场部署了 1000 个传感器，标记为 0-999。
您希望快速确定给定传感器是否已在一小时内 ping 通服务器。

可以使用其键引用当前小时的位图来表示此方案。

*   传感器 123 在 2024 年 1 月 1 日的 00：00 小时内对服务器执行 ping 操作。

<!---->

    > SETBIT pings:2024-01-01-00:00 123 1
    (integer) 0

*   传感器 123 是否在 2024 年 1 月 1 日的 00：00 小时内 ping 服务器？

<!---->

    > GETBIT pings:2024-01-01-00:00 123
    1

*   服务器 456 呢？

<!---->

    > GETBIT pings:2024-01-01-00:00 456
    0

## 基本命令

*   `SETBIT`将提供的偏移处的位设置为 0 或 1。
*   `GETBIT`返回给定偏移处位的值。
*   `BITOP`允许您对一个或多个字符串执行按位运算。

查看[位图命令的完整列表](https://redis.io/commands/?group=bitmap).

## 性能

`SETBIT`和`GETBIT`是 O（1）。
`BITOP`是 O（n），其中*n*是比较中最长字符串的长度。

## 了解更多信息

*   [Redis 位图说明](https://www.youtube.com/watch?v=oj8LdJQjhJo)教您如何在在线游戏中使用位图进行地图探索。
*   [Redis University's RU101](https://university.redis.com/courses/ru101/)详细介绍了 Redis 位图。
