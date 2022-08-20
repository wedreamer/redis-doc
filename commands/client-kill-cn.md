这`CLIENT KILL`命令关闭给定的客户端连接。此命令支持两种格式，即旧格式：

    CLIENT KILL addr:port

这`ip:port`应与 返回的行匹配`CLIENT LIST`命令 （`addr`字段）。

新格式：

    CLIENT KILL <filter> <value> ... ... <filter> <value>

使用新形式，可以通过不同的属性杀死客户端
而不是仅仅通过地址来杀死。可以使用以下筛选器：

*   `CLIENT KILL ADDR ip:port`.这与旧的三参数行为完全相同。
*   `CLIENT KILL LADDR ip:port`.终止连接到指定本地（绑定）地址的所有客户端。
*   `CLIENT KILL ID client-id`.允许通过其唯一属性杀死客户端`ID`田。客户`ID`的 检索使用`CLIENT LIST`命令。
*   `CLIENT KILL TYPE type`哪里*类型*是其中之一`normal`,`master`,`replica`和`pubsub`.这将关闭**所有客户端**在指定的类中。请注意，客户端被阻止到`MONITOR`命令被视为属于`normal`类。
*   `CLIENT KILL USER username`.关闭使用指定值进行身份验证的所有连接[前交叉韧带](/topics/acl)用户名，但是，如果用户名未映射到现有 ACL 用户，则会返回错误。
*   `CLIENT KILL SKIPME yes/no`.默认情况下，此选项设置为`yes`，也就是说，调用该命令的客户端不会被终止，但是将此选项设置为`no`将具有杀死调用命令的客户端的效果。

可以同时提供多个过滤器。该命令将通过逻辑 AND 处理多个筛选器。例如：

    CLIENT KILL addr 127.0.0.1:12345 type pubsub

是有效的，并且只会杀死具有指定地址的 pubsub 客户端。这种包含多个过滤器的格式目前很少有用。

使用新表单时，该命令不再返回`OK`或错误，但相反，已终止的客户端数可能为零。

## Client Kill 和 Redis Sentinel

最新版本的 Redis Sentinel（Redis 2.8.12 或更高版本）使用 CLIENT KILL
为了在重新配置实例时杀死客户端，以便
强制客户端再次与一个哨兵握手并更新
其配置。

## 笔记

由于 Redis 的单线程特性，不可能
在执行命令时终止客户端连接。从
从客户端的角度来看，连接永远无法关闭
在执行命令的过程中。但是，客户端
将注意到连接已关闭，仅当
发送 next 命令（并导致网络错误）。

@return

使用三个参数格式调用时：

@simple字符串回复：`OK`如果连接存在且已关闭

使用过滤器/值格式调用时：

@integer回复：杀死的客户端数。
