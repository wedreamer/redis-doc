---
title: Transactions
linkTitle: Transactions
weight: 1
description: How transactions work in Redis
aliases:
  - /topics/transactions
---

Redis 事务允许执行一组命令
在一个步骤中, 它们以命令为中心
`MULTI`,`EXEC`,`DISCARD`和`WATCH`.
Redis Transactions有两个重要的保证：

*   事务中的所有命令都经过序列化和执行
    顺序。另一个客户端发送的请求永远不会
    上菜**在中间**执行 Redis 事务。
    这保证了命令作为单个命令执行
    隔离操作。

*   这`EXEC`命令
    触发事务中所有命令的执行, 因此
    如果客户端在
    调用之前的事务`EXEC`命令所有操作
    执行, 如果`EXEC`命令被调用, 所有
    执行操作。使用
    [仅追加文件](/topics/persistence#append-only-file)Redis 确保
    使用单个写  (2)  系统调用将事务写入磁盘。
    但是, 如果 Redis 服务器崩溃或被系统管理员杀死
    以某种困难的方式, 可能只有部分数量的操作
    已注册。Redis 将在重新启动时检测到此情况, 并会退出并显示错误。
    使用`redis-check-aof`工具可以修复
    仅追加将删除部分事务的文件, 以便
    服务器可以再次启动。

从版本 2.2 开始, Redis 允许对
以上两种, 以乐观锁定的形式与一种非常相似的方式
检查和设置  (CAS)  操作。
这是有文档记录的[后](#cas)在此页面上。

## 用法

Redis 事务是使用`MULTI`命令。命令
总是回复`OK`.此时, 用户可以发出多个
命令。Redis 将不再执行这些命令, 而是将排队
他们。所有命令执行一次`EXEC`被调用。

叫`DISCARD`而是将刷新事务队列并退出
交易。

以下示例递增键`foo`和`bar`自动。

    > MULTI
    OK
    > INCR foo
    QUEUED
    > INCR bar
    QUEUED
    > EXEC
    1) (integer) 1
    2) (integer) 1

从上面的会议中可以清楚地看出, `EXEC`返回
回复数组, 其中每个元素都是单个命令的回复
在事务中, 以相同的顺序发出命令。

当 Redis 连接位于`MULTI`请求
所有命令都将使用字符串进行回复`QUEUED` (作为状态回复发送
从Redis协议的角度来看) 。排队的命令是
在以下情况下简单地计划执行`EXEC`被调用。

## 事务中的错误

在事务期间, 可能会遇到两种命令错误：

*   命令可能无法排队, 因此之前可能存在错误`EXEC`被调用。
    例如, 该命令可能在语法上是错误的 (错误的参数数量, 
    错误的命令名称, ...) , 或者可能存在一些严重情况, 例如 out
    内存条件 (如果服务器配置为具有内存限制, 则使用`maxmemory`指令) 。
*   命令可能会失败*后* `EXEC`被调用, 例如, 因为我们执行
    对具有错误值的键执行的操作 (如对字符串值调用列表操作) 。

从 Redis 2.6.5 开始, 服务器将在命令累积过程中检测到错误。
然后, 它将拒绝执行事务, 并在`EXEC`, 则放弃事务。

> **Redis < 2.6.5 注释：**在 Redis 2.6.5 之前, 客户端需要检测在`EXEC`通过检查
> 排队命令的返回值：如果命令回复为 QUEUED, 则为
> 已正确排队, 否则 Redis 将返回错误。
> 如果对命令进行排队时出错, 则大多数客户端
> 将中止并丢弃事务。否则, 如果客户选择继续交易
> 这`EXEC`命令将执行所有成功排队的命令, 而不考虑以前的错误。

发生错误*后* `EXEC`相反, 不以特殊方式处理：
所有其他命令将被执行, 即使某些命令在事务期间失败。

这在协议级别上更为明显。在下面的示例 1 中
命令在执行时将失败, 即使语法正确也是如此：

    Trying 127.0.0.1...
    Connected to localhost.
    Escape character is '^]'.
    MULTI
    +OK
    SET a abc
    +QUEUED
    LPOP a
    +QUEUED
    EXEC
    *2
    +OK
    -ERR Operation against a key holding the wrong kind of value

`EXEC`返回的双元素[批量字符串回复](/topics/protocol#bulk-string-reply)其中一个是`OK`代码和
另一个`-ERR`答。由客户端库来查找
向用户提供错误的明智方法。

重要的是要注意
**即使命令失败, 队列中的所有其他命令也会被处理**– Redis will*不*停止
命令的处理。

另一个例子, 再次使用有线协议与`telnet`, 显示了如何
语法错误将尽快报告：

    MULTI
    +OK
    INCR a b c
    -ERR wrong number of arguments for 'incr' command

这次由于语法错误不好`INCR`命令未排队
完全。

## 回滚呢？

Redis 不支持事务回滚, 因为支持回滚
将对 Redis 的简单性和性能产生重大影响。

## 放弃命令队列

`DISCARD`可用于中止事务。在这种情况下, 否
命令被执行, 连接的状态恢复到
正常。

    > SET foo 1
    OK
    > MULTI
    OK
    > INCR foo
    QUEUED
    > DISCARD
    OK
    > GET foo
    "1"

<a name="cas"></a>

## 使用检查和设置的乐观锁定

`WATCH`用于向 Redis 提供检查和设置  (CAS)  行为
交易。

`WATCH`监视 ed 键, 以便检测针对它们的更改。如果
在`EXEC`命令, 
整个事务中止, 以及`EXEC`返回[空回复](/topics/protocol#nil-reply)以通知
事务失败。

例如, 假设我们需要原子递增该值
的键乘以 1 (假设 Redis 没有`INCR`).

第一次尝试可能是：

    val = GET mykey
    val = val + 1
    SET mykey $val

只有当我们有一个客户端执行
在给定时间内操作。如果多个客户端尝试递增密钥
大约在同一时间将出现争用条件。例如
客户端 A 和 B 将读取旧值, 例如 10。该值将
由两个客户端递增到 11, 最后`SET`作为值
的键。因此, 最终值将为 11 而不是 12。

由于`WATCH`我们能够很好地对问题进行建模：

    WATCH mykey
    val = GET mykey
    val = val + 1
    MULTI
    SET mykey $val
    EXEC

使用上述代码, 如果存在争用条件和另一个客户端
修改结果`val`在我们呼叫之间的时间`WATCH`和
我们的呼吁`EXEC`, 则事务将失败。

我们只需要重复这个操作, 希望这次我们不会得到一个
新种族。这种形式的锁定称为*乐观锁定*.
在许多用例中, 多个客户端将访问不同的密钥, 
因此不太可能发生碰撞 - 通常不需要重复操作。

## 手表解释

那么什么是`WATCH`真的是关于吗？这是一个命令, 将
使`EXEC`有条件：我们要求 Redis 执行
仅当没有一个`WATCH`修改了 ed 键。这包括
客户端所做的修改, 如写入命令, 以及 Redis 本身所做的修改, 
比如过期或驱逐。如果键在
`WATCH`ed 和 当`EXEC`收到, 整个交易将被中止
相反。

**注意**

*   在 6.0.9 之前的 Redis 版本中, 过期的密钥不会导致事务
    要中止。[有关此内容的更多信息](https://github.com/redis/redis/pull/7920)
*   事务中的命令不会触发`WATCH`条件, 因为他们
    仅在`EXEC`已发送。

`WATCH`可以多次调用。简单地说, 所有的`WATCH`呼叫将
具有从呼叫开始监视更改的效果, 直到
当下`EXEC`被调用。您还可以将任意数量的密钥发送到
单`WATCH`叫。

什么时候`EXEC`被调用, 所有键都是`UNWATCH`ed, 无论是否
交易是否中止。 此外, 当客户端连接
关闭, 一切都得到`UNWATCH`编辑。

也可以使用`UNWATCH`命令 (不带参数) 
为了刷新所有受监视的键。有时这很有用, 因为我们
乐观地锁定几个键, 因为可能我们需要执行
事务来更改这些键, 但在读取当前内容之后
的键, 我们不想继续。 当这种情况发生时, 我们只需调用
`UNWATCH`因此, 连接已经可以自由地用于新的
交易。

### 使用 WATCH 实现 ZPOP

一个很好的例子来说明如何`WATCH`可用于创建新的
Redis不支持的原子操作是实现ZPOP
(`ZPOPMIN`,`ZPOPMAX`并且仅添加了它们的阻止变体
在 5.0 版中) , 这是一个命令, 用于弹出具有较低值的元素
以原子方式从排序的集合中获取分数。这是最简单的
实现：

    WATCH zset
    element = ZRANGE zset 0 0
    MULTI
    ZREM zset element
    EXEC

如果`EXEC`失败 (即返回[空回复](/topics/protocol#nil-reply)) , 我们只是重复操作。

## Redis 脚本和事务

对于像 redis 中的操作这样的事务, 需要考虑的其他事项是
[redis scripts](/commands/eval)这是事务性的。万事
你可以用Redis事务来做, 你也可以用脚本来做, 以及
通常脚本会更简单, 更快捷。
