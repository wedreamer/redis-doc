此命令可以从其他连接中取消阻止在阻止操作中被阻止的客户端，例如`BRPOP`或`XREAD`或`WAIT`.

默认情况下，客户端处于解除阻止状态，就好像命令的超时值为
已达到，但是，如果传递了其他（和可选）参数，则可以指定取消阻止行为，该行为可以是**超时**（默认值）或**错误**.如果**错误**，则行为是取消阻止客户端，该客户端返回为错误，即客户端已被强制取消阻止。具体而言，客户端将收到以下错误：

    -UNBLOCKED client unblocked via CLIENT UNBLOCK

注意：当然，通常不能保证错误文本仍然存在
相同，但错误代码将保留`-UNBLOCKED`.

此命令很有用，特别是当我们监视许多键时
连接数量有限。例如，我们可能希望监视多个
流`XREAD`不使用超过 N 个连接。然而，在某些
点消费者进程被告知还有一个流键
以进行监视。为了避免使用更多的连接，最好的行为是
要停止来自池中某个连接的阻塞命令，请添加
新密钥，然后再次发出阻止命令。

若要获取此行为，请使用以下模式。该过程使用
一个额外的*控制连接*为了发送`CLIENT UNBLOCK`命令
如果需要。同时，在另一个上运行阻塞操作之前
连接，进程运行`CLIENT ID`为了获得关联的ID
与该连接。何时应添加新密钥，或何时应添加密钥
不再受监视，相关连接阻止命令将中止
通过发送`CLIENT UNBLOCK`在控制连接中。阻止命令
将返回并最终可以重新发行。

但是，此示例在 Redis 流的上下文中显示应用程序
该模式是一般的模式，可以应用于其他情况。

@examples

    Connection A (blocking connection):
    > CLIENT ID
    2934
    > BRPOP key1 key2 key3 0
    (client is blocked)

    ... Now we want to add a new key ...

    Connection B (control connection):
    > CLIENT UNBLOCK 2934
    1

    Connection A (blocking connection):
    ... BRPOP reply with timeout ...
    NULL
    > BRPOP key1 key2 key3 key4 0
    (client is blocked again)

@return

@integer回复，具体而言：

*   `1`如果客户端已成功解除阻止。
*   `0`如果客户端未解除阻止。
