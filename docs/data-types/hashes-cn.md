---
title: "Redis hashes"
linkTitle: "Hashes"
weight: 40
description: >
    Introduction to Redis hashes
---

Redis 哈希是结构化为字段值对集合的记录类型。
您可以使用哈希来表示基本对象并存储计数器分组等。

## 例子

*   将基本用户配置文件表示为哈希：

<!---->

    > HSET user:123 username martina firstName Martina lastName Elisa country GB
    (integer) 4
    > HGET user:123 username
    "martina"
    > HGETALL user:123
    1) "username"
    2) "martina"
    3) "firstName"
    4) "Martina"
    5) "lastName"
    6) "Elisa"
    7) "country"
    8) "GB"

*   存储设备 777 对服务器执行 ping 操作、发出请求或发送错误的次数的计数器：

<!---->

    > HINCRBY device:777:stats pings 1
    (integer) 1
    > HINCRBY device:777:stats pings 1
    (integer) 2
    > HINCRBY device:777:stats pings 1
    (integer) 3
    > HINCRBY device:777:stats errors 1
    (integer) 1
    > HINCRBY device:777:stats requests 1
    (integer) 1
    > HGET device:777:stats pings
    "3"
    > HMGET device:777:stats requests errors
    1) "1"
    2) "1"

## 基本命令

*   `HSET`设置哈希上一个或多个字段的值。
*   `HGET`返回给定字段中的值。
*   `HMGET`返回一个或多个给定字段中的值。
*   `HINCRBY`将给定字段中的值按提供的整数递增。

查看[哈希命令的完整列表](https://redis.io/commands/?group=hash).

## 性能

大多数 Redis 哈希命令都是 O (1) 。

一些命令 - 例如`HKEYS`,`HVALS`和`HGETALL`- 是 O (n) , 其中*n*是字段值对的数量。

## 限制

每个哈希最多可以存储 4, 294, 967, 295  (2^32 - 1)  个字段值对。
实际上, 哈希仅受托管 Redis 部署的 VM 上的总体内存的限制。

## 了解更多信息

*   [Redis Hashes Explained](https://www.youtube.com/watch?v=-KdITaRkQ-U)是一个简短, 全面的视频解释器, 涵盖了 Redis 哈希。
*   [Redis University's RU101](https://university.redis.com/courses/ru101/)详细介绍了 Redis 哈希。
