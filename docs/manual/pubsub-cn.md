---
title: Redis Pub/Sub
linkTitle: "Pub/sub"
weight: 1
description: How to use pub/sub channels in Redis
aliases:
  - /topics/pubsub
  - /docs/manual/pub-sub
---


`SUBSCRIBE`、`UNSUBSCRIBE` 和 `PUBLISH` 实现了 [发布/订阅消息传递范例](http://en.wikipedia.org/wiki/Publish/subscribe), 其中 (引用 Wikipedia) 发件人 (发布者) 未编程为将他们的消息发送给特定的接收者 (订阅者) 。相反, 发布的消息被表征为通道, 而不知道可能有什么 (如果有的话) 订阅者。订阅者对一个或多个信道表示兴趣, 并且只接收感兴趣的消息, 而不知道有什么 (如果有的话) 发布者。发布者和订阅者的这种解耦可以实现更大的可扩展性和更动态的网络拓扑.

例如, 为了订阅信道`foo`和`bar`这客户端发出`SUBSCRIBE`提供通道的名称：

```bash
SUBSCRIBE foo bar
```

其他客户端发送到这些通道的消息将由 Redis 推送添加到所有已订阅的客户端。

订阅了一个或多个通道的客户端不应发出命令, 尽管它可以订阅和取消订阅其他信道。对订阅和取消订阅操作的答复将发送在消息的形式, 以便客户端可以只读取一个连贯的消息流, 其中第一个元素指示消息。在已订阅的上下文中允许的命令客户端是`SUBSCRIBE`,`SSUBSCRIBE`,`SUNSUBSCRIBE`,`PSUBSCRIBE`,`UNSUBSCRIBE`,`PUNSUBSCRIBE`,`PING`,`RESET`和`QUIT`.

请注意：`redis-cli`一旦进入, 将不接受任何命令订阅模式, 并且只能退出模式`Ctrl-C`.

## 推送消息的格式

一个消息是[数组回复](/topics/protocol#array-reply)有三个元素。

第一个元素是消息类型：

*   `subscribe`：表示我们已成功订阅该信道作为答复中的第二个要素给出。第三个参数表示我们当前订阅的信道数量。

*   `unsubscribe`：表示我们已成功取消订阅通道作为回复中的第二个元素给出。第三个参数表示我们当前订阅的信道数。什么时候最后一个参数为零, 我们不再订阅任何通道, 并且客户端可以发出任何类型的 Redis 命令, 因为我们在发布/订阅状态。

*   `message`：它是由于`PUBLISH`命令由另一个客户端发出。第二个元素是原始通道, 第三个参数是实际消息有效载荷。

## 数据库和范围界定

发布/订阅与 key 空间无关。
它被要求不在任何层面上干扰它, 包括数据库编号。

在 db 10 上发布, 将由 db 1 上的订阅者听到。

如果需要某种范围, 请在通道前面加上环境 (测试、暂存、生产...)

## 有线协议示例

```
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
```

此时, 从另一个客户端, 我们发出`PUBLISH`操作针对名为`second`:

    > PUBLISH second Hello

这是第一个客户端接收的内容：

    *3
    $7
    message
    $6
    second
    $5
    Hello

现在, 客户端使用`UNSUBSCRIBE`不带其他参数的命令：

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

## 订阅的模式匹配

Redis Pub/Sub 实现支持模式匹配。客户可以订阅 glob 样式模式以接收所有消息发送到与给定模式匹配的通道名称。

例如：

    PSUBSCRIBE news.*

将接收发送到信道的所有消息`news.art.figurative`, `news.music.jazz`等。
所有 glob 样式的模式都是有效的, 因此支持多个通配符。

    PUNSUBSCRIBE news.*

然后, 将取消订阅客户端的该模式。
此调用不会影响任何其他订阅。

由于模式匹配而接收的消息以不同的格式：

*   消息的类型为`pmessage`：这是收到的消息作为结果`PUBLISH`由另一个客户端发出的命令, 匹配模式匹配订阅。第二个元素是原始元素模式匹配, 第三个元素是原始名称通道, 最后一个元素是实际消息负载。

类似于`SUBSCRIBE`和`UNSUBSCRIBE`,`PSUBSCRIBE`和`PUNSUBSCRIBE`命令由发送消息的系统确认类型`psubscribe`和`punsubscribe`使用与`subscribe`和`unsubscribe`消息格式。

## 与模式和通道订阅匹配的消息

如果客户端已订阅, 则可能会多次收到一条消息到与已发布消息匹配的多个模式, 或者如果是订阅了与消息匹配的模式和通道。喜欢在以下示例：

    SUBSCRIBE foo
    PSUBSCRIBE f*

在上面的示例中, 如果将消息发送到通道`foo`、客户端将收到两条消息：一条类型`message`和类型之一`pmessage`.

## 具有模式匹配的订阅计数的含义

在 `subscribe`、`unsubscribe`、`psubscribe` 和 `punsubscribe` 消息类型中, 最后一个参数是仍然活跃的订阅数。这个数字实际上是客户端仍然订阅的频道和模式的总数。因此, 仅当由于取消订阅所有通道和模式而导致该计数降至零时, 客户端才会退出 Pub/Sub 状态.

## 分片 sub/pub

从 7.0 开始, 引入了分片 Pub/Sub, 其中分片通道通过用于将 key 分配给插槽的相同算法分配给插槽。
分片消息必须发送到拥有分片通道哈希到的插槽的节点。
集群确保将已发布的分片消息转发到分片中的所有节点, 以便客户端可以通过连接到负责插槽的主节点或其任何副本来订阅分片通道。
`SSUBSCRIBE`,`SUNSUBSCRIBE`和`SPUBLISH`用于实现分片发布/订阅。

分片发布/订阅有助于在集群模式下扩展发布/订阅的使用。
它将消息的传播限制在集群的分片内。
因此, 与全局 Pub/Sub 相比, 通过群集总线传递的数据量是有限的, 其中每条消息都传播到群集中的每个节点。
这允许用户通过添加更多分片来水平扩展发布/订阅的使用。

## 编程示例

Pieter Noordhuis 提供了一个使用 EventMachine 的很好的例子和 Redis 创建[多用户高性能网络聊天](https://gist.github.com/pietern/348262).

## 客户端库实现提示

因为接收到的所有消息都包含导致消息传递的原始订阅(消息类型的情况下是通道, 而 pmessage 类型的情况下是原始模式)客户端库可能会将原始订阅绑定到回调(可以是匿名函数, 块, 函数指针), 使用哈希表。

当收到消息时, 可以进行 O(1) 查找, 以便将消息传递给注册的回调。
