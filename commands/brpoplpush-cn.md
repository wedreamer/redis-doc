`BRPOPLPUSH`是`RPOPLPUSH`.
什么时候`source`包含元素, 此命令的行为与`RPOPLPUSH`.
当在`MULTI`/`EXEC`块, 此命令的行为与`RPOPLPUSH`.
什么时候`source`为空, Redis 将阻止连接, 直到另一个客户端
推到它或直到`timeout`已到达。
一个`timeout`的零可用于无限期阻止。

看`RPOPLPUSH`了解更多信息。

@return

@bulk字符串回复：从中弹出的元素`source`并推送到`destination`.
如果`timeout`已到达, 则返回@nil回复。

## 模式：可靠队列

请参阅`RPOPLPUSH`文档。

## 模式：圆形列表

请参阅`RPOPLPUSH`文档。
