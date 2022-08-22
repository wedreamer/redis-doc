---
title: Transactions
linkTitle: Transactions
weight: 1
description: How transactions work in Redis
aliases:
  - /topics/transactions
---

Redis Transactions 允许在一个步骤中执行一组命令，它们以命令 `MULTI`、`EXEC`、`DISCARD` 和 `WATCH` 为中心.
Redis Transactions 有两个重要的保证：

*   事务中的所有命令都被序列化并按顺序执行。另一个客户端发送的请求将永远不会在 Redis 事务执行的**中间**得到服务。

这保证了命令作为单个隔离操作执行。

*   `EXEC` 命令会触发事务中所有命令的执行，因此如果客户端在调用 `EXEC` 命令之前在事务上下文中失去与服务器的连接，则不会执行任何操作，相反，如果 `EXEC`命令被调用，所有的操作都被执行。当使用 [append-only file](/topics/persistence#append-only-file)时, Redis 确保使用单个 write(2) 系统调用将事务写入磁盘。

但是，如果 Redis 服务器崩溃或被系统管理员以某种硬方式杀死，则可能仅注册了部分操作。 Redis 会在重启时检测到这种情况，并会报错退出。
使用 `redis-check-aof` 工具可以修复将删除部分事务的仅附加文件，以便服务器可以重新启动.


从 2.2 版本开始， Redis 允许对上述两个提供额外保证，其形式为乐观锁定，其方式非常类似于 check-and-set (CAS) 操作。
这是有文档记录的[后](#cas)在此页面上。

## 用法

使用 `MULTI` 命令输入 Redis 事务。该命令始终以 "OK" 回复。此时用户可以发出多个命令。 Redis 不会执行这些命令，而是将它们排队。一旦调用 “EXEC”，所有命令都会执行。

调用 `DISCARD` 将刷新事务队列并退出事务。

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

从上面的会话中可以清楚地看出，“EXEC”返回一个回复数组，其中每个元素都是事务中单个命令的回复，与发出命令的顺序相同。

当 Redis 连接在 `MULTI` 请求的上下文中时，所有命令都将使用字符串 `QUEUED` 进行回复（从 Redis 协议的角度来看，作为状态回复发送）。当调用“EXEC”时，队列中的命令只是被安排执行。

## 事务中的错误

在事务期间, 可能会遇到两种命令错误：

* 一个命令可能无法排队，所以在调用 `EXEC` 之前可能会出错。

例如，命令可能在语法上是错误的（参数数量错误，命令名称错误，...），或者可能存在一些严重的情况，例如内存不足的情况（如果服务器配置为使用 ` maxmemory`指令）。

* 一个命令可能会在调用 `EXEC` 之*后*失败，例如，因为我们对具有错误值的键执行了操作（例如对字符串值调用列表操作）.

从 Redis 2.6.5 开始, 服务器将在命令排队过程中检测到错误。
然后, 它将拒绝执行事务, 并在`EXEC`, 则放弃事务。

> **Redis < 2.6.5 的注意事项：** 在 Redis 2.6.5 之前，客户端需要通过检查排队命令的返回值来检测在“执行”之前发生的错误：如果命令以 QUEUED 回复，则表示正确排队，否则 Redis 返回错误。
如果在排队命令时出现错误，大多数客户端将中止并丢弃事务。否则，如果客户端选择继续进行事务，则“EXEC”命令将成功执行所有排队的命令，而不管先前的错误如何。

发生错误*后* `EXEC`相反, 不以特殊方式处理：
所有其他命令将被执行, 即使某些命令在事务期间失败。

这在协议级别上更清楚。在以下示例中，即使语法正确，一个命令也会在执行时失败：

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

`EXEC` 返回了两个元素的 [bulk string reply](/topics/protocol#bulk-string-reply)，其中一个是 `OK` 代码，另一个是 `-ERR` 回复。由客户端库找到一种合理的方式向用户提供错误.

请务必注意，**即使一个命令失败，队列中的所有其他命令也会被处理** – Redis 不会_停止命令的处理。

另一个示例，再次使用带有 `telnet` 的有线协议，显示了如何尽快报告语法错误：

    MULTI
    +OK
    INCR a b c
    -ERR wrong number of arguments for 'incr' command

这次由于语法错误，错误的“INCR”命令根本没有排队。

## 回滚呢？

Redis 不支持事务回滚，因为支持回滚会对 Redis 的简单性和性能产生重大影响。

## 放弃命令队列

`DISCARD` 可用于中止交易。在这种情况下，不执行任何命令，连接状态恢复正常。

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

`WATCH` 用于为 Redis 事务提供检查和设置 (CAS) 行为。

监视“WATCH”键以检测针对它们的更改。如果在 `EXEC` 命令之前至少修改了一个监视键，则整个事务中止，并且 `EXEC` 返回一个 [Null reply](/topics/protocol#nil-reply) 以通知事务失败。

例如，假设我们需要将键的值原子地增加 1（假设 Redis 没有`INCR`）.

第一次尝试可能是：

    val = GET mykey
    val = val + 1
    SET mykey $val

只有当我们有一个客户端在给定时间内执行操作时，这才会可靠地工作。如果多个客户端几乎同时尝试增加 key ，则会出现竞争条件。例如，客户端 A 和 B 将读取旧值，例如 10。该值将由两个客户端递增到 11，最后将 `SET` 作为键的值。所以最终值将是 11 而不是 12。

由于`WATCH`我们能够很好地对问题进行建模：

    WATCH mykey
    val = GET mykey
    val = val + 1
    MULTI
    SET mykey $val
    EXEC

使用上面的代码，如果在我们调用 WATCH 和调用 EXEC 之间存在竞争条件并且另一个客户端修改了 val 的结果，则事务将失败。

我们只需要重复操作，希望这次不会有新的比赛。这种形式的锁定称为_乐观锁定_。
在许多用例中，多个客户端将访问不同的 key ，因此不太可能发生冲突——通常不需要重复操作。

## WATCH 解释

那么“WATCH”到底是关于什么的呢？这是一个使“EXEC”成为条件的命令：我们要求 Redis 仅在没有任何“WATCH”键被修改的情况下才执行事务。这包括客户端所做的修改，如写入命令，以及 Redis 本身所做的修改，如过期或驱逐。如果在“WATCH”和收到“EXEC”之间修改了 key ，则整个事务将被中止。

**注意**

* 在 6.0.9 之前的 Redis 版本中，过期的 key 不会导致事务中止。 [更多内容](https://github.com/redis/redis/pull/7920)。
* 事务中的命令不会触发“WATCH”条件，因为它们仅在发送“EXEC”之前排队.

`WATCH` 可以被多次调用。简单地说，所有的 `WATCH` 调用都将具有观察从调用开始的变化的效果，直到调用 `EXEC` 的那一刻。您还可以将任意数量的键发送到单个“WATCH”调用。

当调用 `EXEC` 时，所有键都是 `UNWATCH`ed，无论事务是否中止。此外，当客户端连接关闭时，所有内容都会“UNWATCH”。

也可以使用 `UNWATCH` 命令（不带参数）来刷新所有被监视的键​​。有时这很有用，因为我们乐观地锁定了一些键，因为可能我们需要执行事务来更改这些键，但是在读取键的当前内容之后，我们不想继续。发生这种情况时，我们只需调用 `UNWATCH` 以便连接可以自由用于新事务。

### 使用 WATCH 实现 ZPOP

一个很好的例子来说明如何使用 `WATCH` 来创建新的原子操作，否则 Redis 不支持是实现 ZPOP（`ZPOPMIN`、`ZPOPMAX` 及其阻塞变体仅在 5.0 版本中添加），这是一个命令以原子方式从排序集中弹出得分较低的元素。这是最简单的实现：

    WATCH zset
    element = ZRANGE zset 0 0
    MULTI
    ZREM zset element
    EXEC

如果`EXEC`失败 (即返回[空回复](/topics/protocol#nil-reply)) , 我们只是重复操作。

## Redis 脚本和事务

redis 中类似事务的操作需要考虑的其他事项是 [redis 脚本](/commands/eval)，它们是事务性的。你可以用 Redis Transaction 做的所有事情，你也可以用一个脚本来做，通常脚本会更简单更快.
