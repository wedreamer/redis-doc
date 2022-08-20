***

## 标题： “Redis Strings”&#xA;链接标题： “字符串”&#xA;体重： 10&#xA;描述： >&#xA;Redis 字符串简介

Redis 字符串存储字节序列，包括文本、序列化对象和二进制数组。
因此，字符串是最基本的 Redis 数据类型。
它们通常用于缓存，但它们支持其他功能，使您可以实现计数器并执行按位操作。

## 例子

*   在 Redis 中存储并检索字符串：

<!---->

    > SET user:1 salvatore
    OK
    > GET user:1
    "salvatore"

*   存储序列化的 JSON 字符串，并将其设置为从现在起 100 秒后过期：

<!---->

    > SET ticket:27 "\"{'username': 'priya', 'ticket_id': 321}\"" EX 100

*   递增计数器：

<!---->

    > INCR views:page:2
    (integer) 1
    > INCRBY views:page:2 10
    (integer) 11

## 限制

默认情况下，单个 Redis 字符串的最大大小为 512 MB。

## 基本命令

### 获取和设置字符串

*   `SET`存储字符串值。
*   `SETNX`仅当键尚不存在时才存储字符串值。对于实现锁很有用。
*   `GET`检索字符串值。
*   `MGET`在单个操作中检索多个字符串值。

### 管理计数器

*   `INCRBY`原子递增（传递负数时递减）存储在给定键处的计数器。
*   存在另一个用于浮点计数器的命令：[INCRBYFLOAT](/commands/incrbyfloat).

### 按位运算

要对字符串执行按位运算，请参阅[位图数据类型](/docs/data-types/bitmaps)文档。

查看[字符串命令的完整列表](/commands/?group=string).

## 性能

大多数字符串操作都是 O（1），这意味着它们非常高效。
但是，请注意`SUBSTR`,`GETRANGE`和`SETRANGE`命令，可以是 O（n）。
这些随机访问字符串命令可能会导致处理大型字符串时出现性能问题。

## 选择

如果要将结构化数据存储为序列化字符串，则可能还需要考虑[Redis 哈希](/docs/data-types/hashes)或[RedisJSON](/docs/stack/json).

## 了解更多信息

*   [Redis Strings Explained](https://www.youtube.com/watch?v=7CUt4yWeRQE)是一个关于Redis字符串的简短，全面的视频解释器。
*   [雷迪斯大学的RU101](https://university.redis.com/courses/ru101/)详细介绍了 Redis 字符串。
