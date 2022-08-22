这`CONFIG SET`命令用于在运行时重新配置服务器
无需重新启动 Redis。
您可以更改两个简单参数, 也可以从一个参数切换到另一个持久性
选项。

支持的配置参数列表`CONFIG SET`可以获得
发出`CONFIG GET *`命令, 即用于获取的对称命令
有关正在运行的 Redis 实例的配置的信息。

所有配置参数设置使用`CONFIG SET`立即加载
由 Redis 执行, 并将从执行下一个命令开始生效。

所有支持的参数都具有与等效参数相同的含义
配置参数用于[redis.conf][hgcarr22rc]文件。

[hgcarr22rc]: http://github.com/redis/redis/raw/unstable/redis.conf

请注意, 您应该查看与您所在的版本相关的 redis.conf 文件
使用 AS 配置选项可能会在版本之间更改。链接
以上是最新的开发版本。

可以将持久性从 RDB 快照切换到仅追加文件
 (反之亦然) 使用`CONFIG SET`命令。
有关如何执行此操作的更多信息, 请查看[坚持
页][tp].

[tp]: /topics/persistence

一般来说, 您应该知道的是, 设置`appendonly`参数
`yes`将启动后台进程以保存初始仅追加文件
 (从内存中数据集获), ) , 并将追加所有后续
命令, 从而获得与
自启动以来在 AOF 打开的情况下启动的 Redis 服务器。

如果需要, 可以使用 RDB 快照同时启用 AOF, 这两者
选项并不相互排斥。

@return

@simple字符串回复：`OK`正确设置配置时。
否则, 将返回错误。
