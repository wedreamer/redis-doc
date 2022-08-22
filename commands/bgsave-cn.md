在后台保存数据库。

通常, 会立即返回 OK 代码。
Redis分叉, 父级继续为客户端服务, 子级保存数据库
然后退出磁盘上。

如果已在后台保存正在运行, 或者如果存在
是另一个正在运行的非后台保存进程, 特别是正在进行的 AOF
重写。

如果`BGSAVE SCHEDULE`使用时, 命令将立即返回`OK`当
AOF 重写正在进行中, 并计划后台保存在下一次运行时运行
机会。

客户端可以使用`LASTSAVE`
命令。

请参阅[持久性文档][tp]以获取详细信息。

[tp]: /topics/persistence

@return

@simple字符串回复：`Background saving started`如果`BGSAVE`已正确启动或`Background saving scheduled`当与`SCHEDULE`子命令。
