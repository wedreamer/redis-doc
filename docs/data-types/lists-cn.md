***

## 标题： “Redis 列表”&#xA;链接标题： “列表”&#xA;体重： 20&#xA;描述： >&#xA;Redis 列表简介

Redis 列表是字符串值的链接列表。
Redis 列表通常用于：

*   实现堆栈和队列。
*   为后台工作线程系统构建队列管理。

## 例子

*   将列表视为队列（先进先出）：

<!---->

    > LPUSH work:queue:ids 101
    (integer) 1
    > LPUSH work:queue:ids 237
    (integer) 2
    > RPOP work:queue:ids
    "101"
    > RPOP work:queue:ids
    "237"

*   将列表视为堆栈（先进先出）：

<!---->

    > LPUSH work:queue:ids 101
    (integer) 1
    > LPUSH work:queue:ids 237
    (integer) 2
    > LPOP work:queue:ids
    "237"
    > LPOP work:queue:ids
    "101"

*   检查列表的长度：

<!---->

    > LLEN work:queue:ids
    (integer) 0

*   从一个列表中原子弹出一个元素并推送到另一个列表：

<!---->

    > LPUSH board:todo:ids 101
    (integer) 1
    > LPUSH board:todo:ids 273
    (integer) 2
    > LMOVE board:todo:ids board:in-progress:ids LEFT LEFT
    "273"
    > LRANGE board:todo:ids 0 -1
    1) "101"
    > LRANGE board:in-progress:ids 0 -1
    1) "273"

*   要创建永远不会超过 100 个元素的上限列表，您可以调用`LTRIM`每次调用后`LPUSH`:

<!---->

    > LPUSH notifications:user:1 "You've got mail!"
    (integer) 1
    > LTRIM notifications:user:1 0 99
    OK
    > LPUSH notifications:user:1 "Your package will be delivered at 12:01 today."
    (integer) 2
    > LTRIM notifications:user:1 0 99
    OK

## 限制

Redis 列表的最大长度为 2^32 - 1 （4，294，967，295） 个元素。

## 基本命令

*   `LPUSH`将新元素添加到列表的头部;`RPUSH`添加到尾巴。
*   `LPOP`从列表的头部删除并返回一个元素;`RPOP`做同样的事情，但从列表的尾巴。
*   `LLEN`返回列表的长度。
*   `LMOVE`以原子方式将元素从一个列表移动到另一个列表。
*   `LTRIM`将列表缩小到指定的元素范围。

### 阻止命令

列表支持多个阻止命令。
例如：

*   `BLPOP`从列表的头部删除并返回一个元素。
    如果该列表为空，则该命令将一直阻止，直到某个元素变为可用或达到指定的超时。
*   `BLMOVE`以原子方式将元素从源列表移动到目标列表。
    如果源列表为空，则该命令将阻塞，直到有新元素变为可用。

查看[完整的列表命令系列](https://redis.io/commands/?group=list).

## 性能

访问其头部或尾部的列表操作是 O（1），这意味着它们非常高效。
但是，操作列表中元素的命令通常是 O（n）。
这些例子包括`LINDEX`,`LINSERT`和`LSET`.
运行这些命令时要小心，尤其是在对大型列表进行操作时。

## 选择

考虑[Redis 流](/docs/data-types/streams)作为列表的替代方法，当您需要存储和处理一系列不确定的事件时。

## 了解更多信息

*   [Redis 列表说明](https://www.youtube.com/watch?v=PB5SeOkkxQc)是 Redis 列表上的一个简短、全面的视频解释器。
*   [雷迪斯大学的RU101](https://university.redis.com/courses/ru101/)详细介绍了 Redis 列表。
