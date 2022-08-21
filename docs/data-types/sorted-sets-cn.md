---
title: "Redis sorted sets"
linkTitle: "Sorted sets"
weight: 50
description: >
    Introduction to Redis sorted sets
---

Redis 排序集是按关联值排序的唯一字符串（成员）的集合。
当多个字符串具有相同的分数时，字符串将按字典顺序排序。
排序集的一些用例包括：

*   排行榜。例如，您可以使用排序集轻松维护大型在线游戏中最高分数的有序列表。
*   速率限制器。特别是，您可以使用排序集来构建滑动窗口速率限制器，以防止过多的 API 请求。

## 例子

*   随着玩家分数的变化更新实时排行榜：

<!---->

    > ZADD leaderboard:455 100 user:1
    (integer) 1
    > ZADD leaderboard:455 75 user:2
    (integer) 1
    > ZADD leaderboard:455 101 user:3
    (integer) 1
    > ZADD leaderboard:455 15 user:4
    (integer) 1
    > ZADD leaderboard:455 275 user:2
    (integer) 0

请注意，`user:2`的分数在决赛中更新`ZADD`叫。

*   获取前 3 名玩家的得分：

<!---->

    > ZRANGE leaderboard:455 0 4 REV WITHSCORES
    1) "user:2"
    2) "275"
    3) "user:3"
    4) "101"
    5) "user:1"
    6) "100"
    7) "user:4"
    8) "15"

*   用户 2 的等级是多少？

<!---->

    > ZREVRANK leaderboard:455 user:2
    (integer) 0

## 基本命令

*   `ZADD`将新成员和关联的分数添加到排序集。如果成员已存在，则更新分数。
*   `ZRANGE`返回在给定范围内排序的已排序集的成员。
*   `ZRANK`返回所提供成员的排名，假定排序按升序排列。
*   `ZREVRANK`返回所提供成员的排名，假定排序集按降序排列。

查看[已排序集命令的完整列表](https://redis.io/commands/?group=sorted-set).

## 性能

大多数排序集操作是 O（log（n）），其中 *n* 是成员数。

在运行 时要小心一些`ZRANGE`具有较大返回值的命令（例如，以数万或更多为单位）。
此命令的时间复杂度为 O（log（n） + m），其中 *m* 是返回的结果数。

## 选择

Redis 排序集有时用于索引其他 Redis 数据结构。
如果需要对数据编制索引和查询，请考虑[RediSearch](/docs/stack/search)和[RedisJSON](/docs/stack/json).

## 了解更多信息

*   [Redis 排序集说明](https://www.youtube.com/watch?v=MUKlxdBQZ7g)是Redis中排序集的有趣介绍。
*   [Redis University's RU101](https://university.redis.com/courses/ru101/)详细探索 Redis 排序集。
