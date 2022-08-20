***

标题： Redis Pub/Sub
链接标题： “Pub/sub”
体重： 1
描述： 如何在 Redis 中使用发布/订阅频道
别名：

*   /topics/pubsub
*   /docs/manual/pub-sub

***

`SUBSCRIBE`,`UNSUBSCRIBE`和`PUBLISH`
实现[发布/订阅消息传递
范例](http://en.wikipedia.org/wiki/Publish/subscribe)哪里
（引用维基百科）发件人（出版商）未被编程为发送
他们向特定接收者（订户）发送的消息。相反，已发布
消息被表征到通道中，而不知道什么（如果
任何）订阅者可能有。订阅者表示对其中一项或
更多的通道，只接收感兴趣的消息，没有
了解有哪些（如果有的话）出版商。这种解耦
发布者和订阅者可以允许更大的可扩展性和更多
动态网络拓扑。

例如，为了订阅频道`foo`和`bar`这
客户端发出`SUBSCRIBE`提供通道的名称：

```bash
SUBSCRIBE foo bar
```

其他客户端发送到这些通道的消息将由 Redis 推送
添加到所有已订阅的客户端。

订阅了一个或多个通道的客户端不应发出命令，
尽管它可以订阅和取消订阅其他频道。
对订阅和取消订阅操作的答复将发送在
消息的形式，以便客户端可以只读取一个连贯的
消息流，其中第一个元素指示
消息。在已订阅的上下文中允许的命令
客户端是`SUBSCRIBE`,`SSUBSCRIBE`,`SUNSUBSCRIBE`,`PSUBSCRIBE`,`UNSUBSCRIBE`,`PUNSUBSCRIBE`,`PING`,`RESET`和`QUIT`.

请注意：`redis-cli`一旦进入，将不接受任何命令
订阅模式，并且只能退出模式`Ctrl-C`.

## 推送消息的格式

消息是[数组回复](/topics/protocol#array-reply)有三个元素。

第一个元素是消息类型：

*   `subscribe`：表示我们已成功订阅该频道
    作为答复中的第二个要素给出。第三个参数表示
    我们当前订阅的频道数量。

*   `unsubscribe`：表示我们已成功取消订阅
    通道作为回复中的第二个元素给出。第三个参数
    表示我们当前订阅的频道数。什么时候
    最后一个参数为零，我们不再订阅任何通道，
    并且客户端可以发出任何类型的 Redis 命令，因为我们在
    发布/订阅状态。

*   `message`：它是由于`PUBLISH`命令
    由另一个客户端发出。第二个元素是
    原始通道，第三个参数是实际消息
    有效载荷。

## 数据库和范围界定

发布/订阅与密钥空间无关。
它被要求不在任何层面上干扰它，包括数据库编号。

在 db 10 上发布，将由 db 1 上的订阅者听到。

如果需要某种范围，请在通道前面加上
环境（测试、暂存、生产...

## 有线协议示例

    SUBSCRIBE first second
    *3
    $9
    subscribe
    $5
    first
    :1
    *3
    $9
    subscribe
    $6
    second
    :2

此时，从另一个客户端，我们发出`PUBLISH`操作
针对名为`second`:

    > PUBLISH second Hello

这是第一个客户端接收的内容：

    *3
    $7
    message
    $6
    second
    $5
    Hello

现在，客户端使用
`UNSUBSCRIBE`不带其他参数的命令：

    UNSUBSCRIBE
    *3
    $11
    unsubscribe
    $6
    second
    :1
    *3
    $11
    unsubscribe
    $5
    first
    :0

## 模式匹配订阅

Redis Pub/Sub 实现支持模式匹配。客户可以
订阅 glob 样式模式以接收所有消息
发送到与给定模式匹配的通道名称。

例如：

    PSUBSCRIBE news.*

将接收发送到频道的所有消息`news.art.figurative`,
`news.music.jazz`等。
所有 glob 样式的模式都是有效的，因此支持多个通配符。

    PUNSUBSCRIBE news.*

然后，将取消订阅客户端的该模式。
此调用不会影响任何其他订阅。

由于模式匹配而接收的消息以
不同的格式：

*   消息的类型为`pmessage`：这是收到的消息
    作为结果`PUBLISH`由另一个客户端发出的命令，匹配
    模式匹配订阅。第二个元素是原始元素
    模式匹配，第三个元素是原始名称
    通道，最后一个元素是实际消息负载。

类似于`SUBSCRIBE`和`UNSUBSCRIBE`,`PSUBSCRIBE`和
`PUNSUBSCRIBE`命令由发送消息的系统确认
类型`psubscribe`和`punsubscribe`使用与
`subscribe`和`unsubscribe`消息格式。

## 与模式和通道订阅匹配的消息

如果客户端已订阅，则可能会多次收到一条消息
到与已发布消息匹配的多个模式，或者如果是
订阅了与消息匹配的模式和通道。喜欢在
以下示例：

    SUBSCRIBE foo
    PSUBSCRIBE f*

在上面的示例中，如果将消息发送到通道`foo`、客户端
将收到两条消息：一条类型`message`和类型之一
`pmessage`.

## 具有模式匹配的订阅计数的含义

在`subscribe`,`unsubscribe`,`psubscribe`和`punsubscribe`
消息类型，最后一个参数是订阅的计数
积极。这个数字实际上是通道的总数，并且
客户端仍订阅的模式。因此，客户端将退出
仅当此计数由于以下原因而降至零时，发布/订阅状态
取消订阅所有渠道和模式。

## 分片酒吧/子

从 7.0 开始，引入了分片 Pub/Sub，其中分片通道通过用于将密钥分配给插槽的相同算法分配给插槽。
分片消息必须发送到拥有分片通道哈希到的插槽的节点。
集群确保将已发布的分片消息转发到分片中的所有节点，以便客户端可以通过连接到负责插槽的主节点或其任何副本来订阅分片通道。
`SSUBSCRIBE`,`SUNSUBSCRIBE`和`SPUBLISH`用于实现分片发布/订阅。

分片发布/订阅有助于在集群模式下扩展发布/订阅的使用。
它将消息的传播限制在集群的分片内。
因此，与全局 Pub/Sub 相比，通过群集总线传递的数据量是有限的，其中每条消息都传播到群集中的每个节点。
这允许用户通过添加更多分片来水平扩展发布/订阅的使用。

## 编程示例

Pieter Noordhuis提供了一个使用EventMachine的很好的例子
和 Redis 创建[多用户高性能网络
聊天](https://gist.github.com/pietern/348262).

## 客户端库实现提示

因为收到的所有消息都包含原始订阅
导致消息传递（消息类型情况下的通道，
和在 pmessage 类型的情况下的原始模式）客户端库
可以将原始订阅绑定到回调（可以是匿名的）
函数、块、函数指针），使用哈希表。

当收到消息时，可以执行 O（1） 查找，以便
将消息传递到已注册的回调。
