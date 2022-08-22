---
title: "Redis sets"
linkTitle: "Sets"
weight: 30
description: >
    Introduction to Redis sets
---

Redis 集是唯一字符串 (成员) 的无序集合。
您可以使用 Redis 集有效地：

*   跟踪唯一项目 (例如, 跟踪访问给定博客文章的所有唯一 IP 地址) 。
*   表示关系 (例如, 具有给定角色的所有用户的集合) 。
*   执行常见的集合运算, 如交集、并集和差值。

## 例子

*   为用户 123 和 456 存储收藏的书籍 ID 集：

<!---->

    > SADD user:123:favorites 347
    (integer) 1
    > SADD user:123:favorites 561
    (integer) 1
    > SADD user:123:favorites 742
    (integer) 1
    > SADD user:456:favorites 561
    (integer) 1

*   检查用户 123 是否喜欢图书 742 和 299

<!---->

    > SISMEMBER user:123:favorites 742
    (integer) 1
    > SISMEMBER user:123:favorites 299
    (integer) 0

*   用户 123 和 456 是否有任何共同喜欢的书籍？

<!---->

    > SINTER user:123:favorites user:456:favorites
    1) "561"

*   用户 123 收藏了多少本书？

<!---->

    > SCARD user:123:favorites
    (integer) 3

## 限制

Redis 集的最大大小为 2^32 - 1  (4, 294, 967, 295)  个成员。

## 基本命令

*   `SADD`将新成员添加到集合中。
*   `SREM`从集合中删除指定的成员。
*   `SISMEMBER`测试字符串的集成员身份。
*   `SINTER`返回两个或多个集合共有的成员集 (即交集) 。
*   `SCARD`返回集合的大小 (又名基数) 。

查看[设置命令的完整列表](https://redis.io/commands/?group=set).

## 性能

大多数集合操作 (包括添加、删除和检查项目是否为集合成员) 都是 O (1) , 这意味着它们非常高效。
但是, 对于具有数十万或更多成员的大型集合, 在运行 `SMEMBERS` 命令。
此命令为 O (n) , 并在单个响应中返回整个集合。
作为替代方案, 请考虑 `SSCAN`, 允许您以迭代方式检索集合的所有成员。

## 选择

对大型数据集 (或流数据) 设置成员资格检查可能会占用大量内存。
如果您担心内存使用情况并且不需要完美的精度, 请考虑[布隆过滤器或布谷鸟过滤器](/docs/stack/bloom)作为集合的替代项。

Redis 集经常被用作一种索引。
如果需要对数据编制索引和查询, 请考虑[RediSearch](/docs/stack/search)] (/docs/stack/search)  和[RedisJSON](/docs/stack/json).

## 了解更多信息

*   [Redis 集说明](https://www.youtube.com/watch?v=PKdCppSNTGQ)和[Redis 套装精心设计](https://www.youtube.com/watch?v=aRw5ME\_5kMY)是两个简短但全面的视频解释器, 涵盖了Redis集。
*   [Redis University's RU101](https://university.redis.com/courses/ru101/)详细探索 Redis 集。
