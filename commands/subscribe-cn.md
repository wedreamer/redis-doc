为客户端订阅指定的通道。

一旦客户端进入订阅状态，它就不应该发出任何
其他命令，附加命令除外`SUBSCRIBE`,`SSUBSCRIBE`,`PSUBSCRIBE`,`UNSUBSCRIBE`,`SUNSUBSCRIBE`,
`PUNSUBSCRIBE`,`PING`,`RESET`和`QUIT`命令。

## 行为更改历史记录

*   `>= 6.2.0`:`RESET`可以调用以退出订阅状态。
