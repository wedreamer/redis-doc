此命令执行连接的服务器端上下文的完全重置, 
模仿断开连接并再次重新连接的效果。

当从常规客户端连接调用该命令时, 它会执行
以后：

*   丢弃电流`MULTI`事务块 (如果存在) 。
*   取消监视所有键`WATCH`由连接完成。
*   禁用`CLIENT TRACKING`, 如果正在使用中。
*   将连接设置为`READWRITE`模式。
*   取消连接的`ASKING`模式 (如果之前已设) ) 。
*   集`CLIENT REPLY`自`ON`.
*   将协议版本设置为 RESP2。
*   `SELECT`s 数据库 0.
*   出口`MONITOR`模式, 如果适用。
*   中止 Pub/Sub 的订阅状态  (`SUBSCRIBE`和`PSUBSCRIBE), ) , 当
    适当。
*   取消连接, 需要呼叫`AUTH`在以下情况下重新进行身份验证
    身份验证已启用。

@return

@simple字符串回复：始终“重置”。
