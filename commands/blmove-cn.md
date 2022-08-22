`BLMOVE`是`LMOVE`.
什么时候`source`包含元素, 此命令的行为与`LMOVE`.
当在`MULTI`/`EXEC`块, 此命令的行为与`LMOVE`.
什么时候`source`为空, Redis 将阻止连接, 直到另一个客户端
推到它或直到`timeout` (指定要阻止的最大秒数的双精度值) 。
一个`timeout`的零可用于无限期阻止。

此命令将替换现已弃用的`BRPOPLPUSH`.行为
`BLMOVE RIGHT LEFT`是等效的。

看`LMOVE`了解更多信息。

@return

@bulk字符串回复：从中弹出的元素`source`并推送到`destination`.
如果`timeout`已到达, 则返回@nil回复。

## 模式：可靠队列

请参阅`LMOVE`文档。

## 模式：圆形列表

请参阅`LMOVE`文档。
