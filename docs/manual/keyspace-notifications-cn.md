***

## 标题：“Redis 键空间通知”&#xA;链接标题： “键空间通知”&#xA;体重： 1&#xA;描述： >&#xA;实时监控对 Redis 键和值的更改&#xA;别名：&#xA;\- /主题/通知

键空间通知允许客户端按顺序订阅发布/订阅通道
以某种方式接收影响 Redis 数据集的事件。

可以接收的事件示例包括：

*   影响给定键的所有命令。
*   接收 LPUSH 操作的所有密钥。
*   数据库中所有即将过期的密钥 0.

注意：Redis Pub/Sub 是*火与忘*也就是说，如果您的发布/订阅客户端断开连接，
并在以后重新连接，在客户端期间传递的所有事件
断开连接将丢失。

### 事件类型

键空间通知是通过发送两种不同类型的事件来实现的
对于影响 Redis 数据空间的每个操作。例如`DEL`
以名为`mykey`在数据库中`0`将触发
传递两条消息，完全等效于以下两条消息
`PUBLISH`命令：

    PUBLISH __keyspace@0__:mykey del
    PUBLISH __keyevent@0__:del mykey

第一个通道侦听所有事件定位
关键`mykey`而另一个频道只侦听`del`操作
键上的事件`mykey`

第一类活动，用`keyspace`频道中的前缀称为
一个**键空间通知**，而第二个，与`keyevent`前缀
称为**关键事件通知**.

在前面的示例中，一个`del`为密钥生成事件`mykey`导致
在两条消息中：

*   密钥空间通道以消息形式接收事件的名称。
*   密钥事件通道以消息形式接收密钥的名称。

可以只启用一种通知才能交付
只是我们感兴趣的事件的子集。

### 配置

默认情况下，键空间事件通知处于禁用状态，因为虽然没有
非常明智的功能使用一些CPU功率。通知已启用
使用`notify-keyspace-events`的 redis.conf 或通过**配置集**.

将参数设置为空字符串将禁用通知。
为了启用该功能，使用非空字符串，由多个组成
字符，其中每个字符都具有特殊含义，具体取决于
下表：

    K     Keyspace events, published with __keyspace@<db>__ prefix.
    E     Keyevent events, published with __keyevent@<db>__ prefix.
    g     Generic commands (non-type specific) like DEL, EXPIRE, RENAME, ...
    $     String commands
    l     List commands
    s     Set commands
    h     Hash commands
    z     Sorted set commands
    t     Stream commands
    d     Module key type events
    x     Expired events (events generated every time a key expires)
    e     Evicted events (events generated when a key is evicted for maxmemory)
    m     Key miss events (events generated when a key that doesn't exist is accessed)
    n     New key events (Note: not included in the 'A' class)
    A     Alias for "g$lshztxed", so that the "AKE" string means all the events except "m".

至少`K`或`E`应存在于字符串中，否则没有事件
无论字符串的其余部分如何，都将传递。

例如，要仅为列表启用键空间事件，则配置
参数必须设置为`Kl`，依此类推。

您可以使用字符串`KEA`以启用大多数类型的事件。

### 不同命令生成的事件

不同的命令根据以下列表生成不同类型的事件。

*   `DEL`生成一个`del`每个已删除密钥的事件。
*   `RENAME`生成两个事件，`rename_from`事件，以及`rename_to`目标键的事件。
*   `MOVE`生成两个事件，`move_from`事件，以及`move_to`目标键的事件。
*   `COPY`生成一个`copy_to`事件。
*   `MIGRATE`生成一个`del`事件（如果删除了源密钥）。
*   `RESTORE`生成一个`restore`事件。
*   `EXPIRE`及其所有变体（`PEXPIRE`,`EXPIREAT`,`PEXPIREAT`） 生成一个`expire`事件，当调用时具有正超时（或将来的时间戳）。请注意，当调用这些命令时，过去的超时值为负值或时间戳时，将删除该键，并且仅`del`而是生成事件。
*   `SORT`生成一个`sortstore`事件时`STORE`用于设置新键。如果结果列表为空，并且`STORE`选项已使用，并且已经存在具有该名称的现有密钥，结果是该密钥已被删除，因此`del`在这种情况下生成事件。
*   `SET`及其所有变体（`SETEX`,`SETNX`,`GETSET`） 生成`set`事件。然而`SETEX`还将生成一个`expire`事件。
*   `MSET`生成一个单独的`set`每个键的事件。
*   `SETRANGE`生成一个`setrange`事件。
*   `INCR`,`DECR`,`INCRBY`,`DECRBY`命令全部生成`incrby`事件。
*   `INCRBYFLOAT`生成一个`incrbyfloat`事件。
*   `APPEND`生成一个`append`事件。
*   `LPUSH`和`LPUSHX`生成单个`lpush`事件，即使在可变参数情况下也是如此。
*   `RPUSH`和`RPUSHX`生成单个`rpush`事件，即使在可变参数情况下也是如此。
*   `RPOP`生成一个`rpop`事件。此外`del`如果由于弹出列表中的最后一个元素而删除键，则会生成事件。
*   `LPOP`生成一个`lpop`事件。此外`del`如果由于弹出列表中的最后一个元素而删除键，则会生成事件。
*   `LINSERT`生成一个`linsert`事件。
*   `LSET`生成一个`lset`事件。
*   `LREM`生成一个`lrem`事件，以及另外一个`del`事件，如果结果列表为空并且该键已被删除。
*   `LTRIM`生成一个`ltrim`事件，以及另外一个`del`事件，如果结果列表为空并且该键已被删除。
*   `RPOPLPUSH`和`BRPOPLPUSH`生成一个`rpop`事件和`lpush`事件。在这两种情况下，订单都是有保证的（`lpush`事件将始终在`rpop`事件）。此外`del`如果生成的列表长度为零且删除了键，则将生成事件。
*   `LMOVE`和`BLMOVE`生成一个`lpop`/`rpop`事件（取决于 wherefrom 参数）和`lpush`/`rpush`事件（取决于 whereto 参数）。在这两种情况下，订单都是有保证的（`lpush`/`rpush`事件将始终在`lpop`/`rpop`事件）。此外`del`如果生成的列表长度为零且删除了键，则将生成事件。
*   `HSET`,`HSETNX`和`HMSET`全部生成一个`hset`事件。
*   `HINCRBY`生成一个`hincrby`事件。
*   `HINCRBYFLOAT`生成一个`hincrbyfloat`事件。
*   `HDEL`生成单个`hdel`事件，以及一个附加事件`del`事件，如果生成的哈希为空并且密钥已被删除。
*   `SADD`生成单个`sadd`事件，即使在可变参数情况下也是如此。
*   `SREM`生成单个`srem`事件，以及一个附加事件`del`事件，如果结果集为空并且删除了键。
*   `SMOVE`生成一个`srem`事件，以及`sadd`目标键的事件。
*   `SPOP`生成一个`spop`事件，以及一个附加事件`del`事件，如果结果集为空并且删除了键。
*   `SINTERSTORE`,`SUNIONSTORE`,`SDIFFSTORE`生成`sinterstore`,`sunionstore`,`sdiffstore`事件分别。在特殊情况下，结果集为空，并且存储结果的键已存在，`del`事件是在删除密钥后生成的。
*   `ZINCR`生成一个`zincr`事件。
*   `ZADD`生成单个`zadd`事件，即使添加了多个元素也是如此。
*   `ZREM`生成单个`zrem`事件，即使删除了多个元素也是如此。当生成的排序集为空并生成键时，将附加一个`del`将生成事件。
*   `ZREMBYSCORE`生成单个`zrembyscore`事件。当生成的排序集为空并生成键时，将附加一个`del`将生成事件。
*   `ZREMBYRANK`生成单个`zrembyrank`事件。当生成的排序集为空并生成键时，将附加一个`del`将生成事件。
*   `ZDIFFSTORE`,`ZINTERSTORE`和`ZUNIONSTORE`分别生成`zdiffstore`,`zinterstore`和`zunionstore`事件。在特殊情况下，生成的排序集为空，并且存储结果的键已存在，一个`del`事件是在删除密钥后生成的。
*   `XADD`生成一个`xadd`事件，可能遵循`xtrim`事件，当与`MAXLEN`子命令。
*   `XDEL`生成单个`xdel`事件，即使删除了多个条目。
*   `XGROUP CREATE`生成一个`xgroup-create`事件。
*   `XGROUP CREATECONSUMER`生成一个`xgroup-createconsumer`事件。
*   `XGROUP DELCONSUMER`生成一个`xgroup-delconsumer`事件。
*   `XGROUP DESTROY`生成一个`xgroup-destroy`事件。
*   `XGROUP SETID`生成一个`xgroup-setid`事件。
*   `XSETID`生成一个`xsetid`事件。
*   `XTRIM`生成一个`xtrim`事件。
*   `PERSIST`生成一个`persist`事件（如果已成功删除与密钥关联的到期时间）。
*   每当与生存时间关联的密钥因过期而从数据集中删除时，一个`expired`将生成事件。
*   每次从数据集中逐出密钥以释放内存时，由于`maxmemory`策略，`evicted`将生成事件。
*   每当向数据集添加新密钥时，都会有`new`将生成事件。

**重要**仅当真正修改了目标键时，所有命令才会生成事件。例如`SREM`从 Set 中删除不存在的元素实际上不会更改键的值，因此不会生成任何事件。

如果对给定命令的事件生成方式有疑问，最简单的
要做的是观察自己：

    $ redis-cli config set notify-keyspace-events KEA
    $ redis-cli --csv psubscribe '__key*__:*'
    Reading messages... (press Ctrl-C to quit)
    "psubscribe","__key*__:*",1

此时使用`redis-cli`在另一个终端中将命令发送到
Redis 服务器并观察生成的事件：

    "pmessage","__key*__:*","__keyspace@0__:foo","set"
    "pmessage","__key*__:*","__keyevent@0__:set","foo"
    ...

### 过期事件的时间

Redis 以两种方式过期具有生存时间关联的密钥：

*   当命令访问密钥并发现该密钥已过期时。
*   通过后台系统，在后台以增量方式查找过期的密钥，以便能够收集从未访问过的密钥。

这`expired`事件是在上述系统之一访问密钥并发现已过期时生成的，因此无法保证 Redis 服务器能够生成`expired`事件在关键时间时到达零值。

如果没有命令持续针对密钥，并且有许多密钥关联了 TTL，则在密钥活动时间降至零的时间与`expired`将生成事件。

基本上`expired`事件**在 Redis 服务器删除密钥时生成**而不是当理论上生活的时间达到零值时。

### 群集中的事件

Redis 集群的每个节点都会生成有关其自己的密钥空间子集的事件，如上所述。但是，与群集中的常规发布/订阅通信不同，事件通知**不是**广播到所有节点。换句话说，键空间事件是特定于节点的。这意味着要接收集群的所有键空间事件，客户端需要订阅每个节点。

@history

*   `>= 6.0`：添加了关键未命中事件。
*   `>= 7.0`：事件类型`new`添加
