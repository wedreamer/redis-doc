---
title: "Redis configuration"
linkTitle: "Configuration"
weight: 1
description: >
    Overview of redis.conf, the Redis configuration file
aliases:
    - /docs/manual/config
---


Redis 能够在没有配置文件的情况下使用内置的默认值启动配置, 但仅建议将此设置用于测试和开发目的。

配置 Redis 的正确方法是提供 Redis 配置文件, 通常称为`redis.conf`.

这`redis.conf`file 包含许多指令, 这些指令具有非常简单的格式：

    keyword argument1 argument2 ... argumentN

这是配置指令的一个示例：

    replicaof 127.0.0.1 6380

可以使用以下命令提供包含空格的字符串作为参数 (双引号或单引号) , 如以下示例所示：

    requirepass "hello world"

单引号字符串可以包含由反斜杠转义的字符, 以及双引号字符串还可以包含使用反斜杠十六进制表示法 "\xff"。

配置指令列表及其含义和预期用法在自有文档的示例 redis.conf 中可用, 该示例已装入Redis distribution.

*   记录在[redis.conf for Redis 7.0](https://raw.githubusercontent.com/redis/redis/7.0/redis.conf).
*   记录在[redis.conf for Redis 6.2](https://raw.githubusercontent.com/redis/redis/6.2/redis.conf).
*   记录在[redis.conf for Redis 6.0](https://raw.githubusercontent.com/redis/redis/6.0/redis.conf).
*   记录在[redis.conf for Redis 5.0](https://raw.githubusercontent.com/redis/redis/5.0/redis.conf).
*   记录在[redis.conf for Redis 4.0](https://raw.githubusercontent.com/redis/redis/4.0/redis.conf).
*   记录在[redis.conf for Redis 3.2](https://raw.githubusercontent.com/redis/redis/3.2/redis.conf).
*   记录在[redis.conf for Redis 3.0](https://raw.githubusercontent.com/redis/redis/3.0/redis.conf).
*   记录在[redis.conf for Redis 2.8](https://raw.githubusercontent.com/redis/redis/2.8/redis.conf).
*   记录在[redis.conf for Redis 2.6](https://raw.githubusercontent.com/redis/redis/2.6/redis.conf).
*   记录在[redis.conf for Redis 2.4](https://raw.githubusercontent.com/redis/redis/2.4/redis.conf).

## 通过命令行传递参数

您还可以传递 Redis 配置参数直接使用命令行。这对于测试目的非常有用。下面是使用端口 6380 启动新的 Redis 实例的示例作为在 127.0.0.1 端口 6379 上运行的实例的副本。

    ./redis-server --port 6380 --replicaof 127.0.0.1 6379

通过命令行传递的参数格式与 redis.conf 文件中使用的格式完全相同, 不同之处在于关键字以 `--` 为前缀.

请注意, 这在内部会生成一个内存中的临时配置文件(可能连接用户传递的配置文件, 如果有的话), 其中参数被转换为 redis.conf 的格式。

## 在服务器运行时更改 Redis 配置

可以动态重新配置 Redis, 而无需停止和重新启动服务, 或使用特殊命令`CONFIG SET`和`CONFIG GET`.

并非所有配置指令都以这种方式受支持, 但大多数按预期方式受支持。请参阅`CONFIG SET`和`CONFIG GET`页以获取更多信息。

请注意, 即时修改配置**对 redis.conf 文件没有影响**, 因此在下次重新启动 Redis 时, 将使用旧配置代替。

确保还根据您使用 `CONFIG SET` 设置的配置修改 `redis.conf` 文件。
您可以手动执行, 也可以使用 `CONFIG REWRITE`, 它会自动扫描您的 `redis.conf` 文件并更新与当前配置值不匹配的字段。
不添加不存在但设置为默认值的字段。
保留配置文件中的注释。

## 将 Redis 配置为缓存

如果您计划将 Redis 用作缓存, 其中每个 key 都有一个过期集, 可以考虑改用以下配置 (以最大内存限制为 2 MB 为例) ：

    maxmemory 2mb
    maxmemory-policy allkeys-lru

在此配置中, 应用程序无需使用 "EXPIRE" 命令 (或等效命令) 为 key 设置生存时间, 因为只要我们达到 2 兆字节的内存限制, 所有密钥都将使用近似的 LRU 算法被驱逐。

基本上, 在这种配置中, Redis 的行为方式与 memcached 类似。
我们有更多关于使用 Redis 作为 LRU 缓存的文档[这里](/topics/lru-cache).
