这`CONFIG GET`命令用于读取
正在运行 Redis 服务器。
并非所有配置参数在 Redis 2.4 中都受支持，而 Redis 2.6 支持
可以使用此命令读取服务器的整个配置。

用于在运行时更改配置的对称命令是`CONFIG
SET`.

`CONFIG GET`采用多个参数，这些参数是 glob 样式的模式。
与任何模式匹配的任何配置参数都将报告为列表
键值对。
例：

    redis> config get *max-*-entries* maxmemory
     1) "maxmemory"
     2) "0"
     3) "hash-max-listpack-entries"
     4) "512"
     5) "hash-max-ziplist-entries"
     6) "512"
     7) "set-max-intset-entries"
     8) "512"
     9) "zset-max-listpack-entries"
    10) "128"
    11) "zset-max-ziplist-entries"
    12) "128"

您可以通过键入以下内容来获取所有受支持配置参数的列表
`CONFIG GET *`在开放中`redis-cli`提示。

所有支持的参数都具有与等效参数相同的含义
配置参数用于[redis.conf][hgcarr22rc]文件：

[hgcarr22rc]: http://github.com/redis/redis/raw/unstable/redis.conf

请注意，您应该查看与您所在的版本相关的 redis.conf 文件
使用 AS 配置选项可能会在版本之间更改。链接
以上是最新的开发版本。

@return

该命令的返回类型是@array回复。
