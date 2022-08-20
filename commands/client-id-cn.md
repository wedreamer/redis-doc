该命令仅返回当前连接的 ID。每个连接
ID有一定的保证：

1.  它永远不会重复，所以如果`CLIENT ID`返回相同的号码，调用方可以确定底层客户端没有断开并重新连接连接，但它仍然是相同的连接。
2.  ID 是单调增量的。如果一个连接的 ID 大于另一个连接的 ID，则保证以后与服务器建立第二个连接。

此命令与`CLIENT UNBLOCK`这是
在 Redis 5 中也与`CLIENT ID`.检查`CLIENT UNBLOCK`涉及这两个命令的模式的命令页。

@examples

```cli
CLIENT ID
```

@return

@integer回复

客户端的 ID。
