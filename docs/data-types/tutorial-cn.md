***

## 标题： “Redis 数据类型教程”&#xA;链接标题： “教程”&#xA;描述： 学习基本的 Redis 数据类型以及如何使用它们&#xA;体重： 1&#xA;别名：&#xA;\- /主题/数据类型介绍&#xA;\- /docs/manual/data-types/data-types-tutorial

下面是一个实践教程，介绍如何使用 Redis CLI 的核心 Redis 数据类型。有关数据类型的一般概述，请参阅[数据类型介绍](/docs/data-types/).

## 钥匙

Redis 密钥是二进制安全的，这意味着您可以使用任何二进制序列作为
键，从“foo”等字符串到 JPEG 文件的内容。
空字符串也是一个有效的键。

关于键的其他一些规则：

*   很长的钥匙不是一个好主意。例如，1024字节的密钥是坏的
    想法不仅在记忆方面，而且因为查找密钥在
    数据集可能需要几个代价高昂的密钥比较。即使任务就在眼前
    是匹配大值的存在，对其进行哈希处理（例如
    与SHA1）是一个更好的主意，特别是从内存的角度来看
    和带宽。
*   非常短的键通常不是一个好主意。写作没有多大意义
    “u1000flw”作为键，如果您可以改为“user：1000：followers”。 后者
    更具可读性，与使用的空间相比，增加的空间很小
    键对象本身和值对象。虽然短键显然会
    消耗的内存少一点，你的工作就是找到合适的平衡点。
*   尝试坚持使用架构。例如，“object-type：id”是一个很好的
    想法，如“用户：1000”。点或破折号通常用于多词
    字段，如“comment：4321：reply.to”或“comment：4321：reply-to”。
*   允许的最大密钥大小为 512 MB。

<a name="strings"></a>

## 字符串

Redis 字符串类型是可以与之关联的最简单的值类型
一个 Redis 密钥。它是Memcached中唯一的数据类型，因此它也非常自然
供新手在Redis中使用它。

由于 Redis 键是字符串，因此当我们也将字符串类型用作值时，
我们正在将一个字符串映射到另一个字符串。字符串数据类型很有用
适用于许多用例，例如缓存 HTML 片段或页面。

让我们玩一下字符串类型，使用`redis-cli`（所有示例
将通过以下方式执行`redis-cli`在本教程中）。

    > set mykey somevalue
    OK
    > get mykey
    "somevalue"

如您所见，使用`SET`和`GET`命令是我们设置的方式
并检索字符串值。请注意，`SET`将替换任何现有值
已存储到密钥中，如果密钥已存在，即使
键与非字符串值相关联。所以`SET`执行分配。

值可以是各种类型的字符串（包括二进制数据），例如
可以将 jpeg 图像存储在值内。值不能大于 512 MB。

这`SET`命令具有有趣的选项，这些选项作为附加选项提供
参数。例如，我可能会问`SET`如果密钥已存在，则失败，
或者相反，仅当密钥已存在时，它才会成功：

    > set mykey newval nx
    (nil)
    > set mykey newval xx
    OK

即使字符串是 Redis 的基本值，也有有趣的操作
你可以和他们一起表演。例如，一个是原子增量：

    > set counter 100
    OK
    > incr counter
    (integer) 101
    > incr counter
    (integer) 102
    > incrby counter 50
    (integer) 152

这[国际唱片业协会](/commands/incr)命令将字符串值解析为整数，
将其递增 1，最后将获得的值设置为新值。
还有其他类似的命令，如[因克比](/commands/incrby),
[断续器](/commands/decr)和[德克里](/commands/decrby).在内部
总是相同的命令，以稍微不同的方式行事。

INCR是原子的是什么意思？
即使有多个客户端发布 INCR
同一密钥永远不会进入争用状态。例如，它永远不会
碰巧客户端 1 读取“10”，客户端 2 同时读取“10”，两者
递增为 11，并将新值设置为 11。最终值将始终为
12 和读取增量集操作执行，而所有其他
客户端不会同时执行命令。

有许多用于操作字符串的命令。例如
这`GETSET`命令将键设置为新值，将旧值作为
结果。您可以使用此命令，例如，如果您有
系统，该系统使用`INCR`
每次您的网站收到新访问者时。你可能想收集这个
信息每小时一次，而不会丢失任何增量。
您可以`GETSET`键，为其分配新值“0”并读取
旧值回来了。

能够在单个密钥中设置或检索多个键的值
命令对于减少延迟也很有用。因此，有
这`MSET`和`MGET`命令：

    > mset a 10 b 20 c 30
    OK
    > mget a b c
    1) "10"
    2) "20"
    3) "30"

什么时候`MGET`，Redis 返回一个值数组。

## 更改和查询密钥空间

有些命令没有在特定类型上定义，但很有用
为了与键的空间相互作用，因此，可以使用
任何类型的键。

例如`EXISTS`命令返回 1 或 0 以指示给定键
数据库中存在或不存在，而`DEL`命令删除密钥
和关联的值，无论值是什么。

    > set mykey hello
    OK
    > exists mykey
    (integer) 1
    > del mykey
    (integer) 1
    > exists mykey
    (integer) 0

从示例中，您还可以了解如何`DEL`本身返回 1 或 0，具体取决于是否
密钥已被删除（它存在）或不存在（没有这样的密钥
名称）。

有很多与键空间相关的命令，但以上两个是
基本与`TYPE`命令，返回种类
存储在指定键处的值：

    > set mykey x
    OK
    > type mykey
    string
    > del mykey
    (integer) 1
    > type mykey
    none

## 密钥过期

在继续之前，我们应该看一个重要的 Redis 功能，无论您存储哪种类型的值，它都可以工作：密钥过期。密钥过期允许您设置密钥的超时，也称为“生存时间”或“TTL”。当生存时间过去时，密钥将自动销毁。

有关密钥过期的一些重要注意事项：

*   可以使用秒或毫秒精度进行设置。
*   但是，过期时间分辨率始终为 1 毫秒。
*   有关过期的信息将被复制并保留在磁盘上，当 Redis 服务器保持停止状态时，该时间几乎过去了（这意味着 Redis 会保存密钥的过期日期）。

使用`EXPIRE`命令来设置密钥的过期时间：

    > set key some-value
    OK
    > expire key 5
    (integer) 1
    > get key (immediately)
    "some-value"
    > get key (after some time)
    (nil)

钥匙在两者之间消失了`GET`调用，因为第二个调用是
延迟超过 5 秒。在上面的例子中，我们使用了`EXPIRE`在
订单设置过期（也可以使用，以便设置不同的
过期到已有密钥的密钥，例如`PERSIST`可按顺序使用
以删除过期并使密钥永久保留）。但是我们
还可以使用其他 Redis 命令创建具有过期的密钥。例如
用`SET`选项：

    > set key 100 ex 10
    OK
    > ttl key
    (integer) 9

上面的示例使用字符串值设置键`100`，具有过期
十秒。后来`TTL`命令被调用以检查
剩余时间为密钥生活。

要设置和检查以毫秒为单位过期，请检查`PEXPIRE`和
这`PTTL`命令，以及的完整列表`SET`选项。

<a name="lists"></a>

## 列表

要解释List数据类型，最好从一点理论开始，
作为术语*列表*经常被信息技术以不正当的方式使用
人。例如，“Python列表”不是名称可能暗示的（链接
列表），而是数组（相同的数据类型称为数组
实际上，红宝石）。

从非常一般的角度来看，列表只是一个有序的序列
元素：10，20，1，2，3 是一个列表。但是 List 的属性是使用
数组与使用
*链表*.

Redis 列表是通过链接列表实现的。这意味着即使您有
列表中的数百万个元素，在
执行列表的头部或尾部*在恒定时间内*.添加的速度
具有`LPUSH`命令到具有十个的列表的头部
元素与将元素添加到列表开头（10 个）相同
百万个元素。

缺点是什么？访问元素*按索引*在列表中非常快
使用数组（恒定时间索引访问）实现，并且不是那么快
由链接列表实现的列表（其中操作需要一定数量的
工作与被访问元素的索引成比例）。

Redis Lists是用链表实现的，因为对于数据库系统来说，它
对于能够以非常快速的方式将元素添加到非常长的列表中至关重要。
正如您稍后将看到的，另一个强大的优势是Redis Lists可以
在恒定的时间内以恒定的长度拍摄。

当快速访问大型元素集合的中间很重要时，
可以使用不同的数据结构，称为排序集。
本教程稍后将介绍排序集。

### 使用 Redis 列表的第一步

这`LPUSH`命令将新元素添加到列表中，在
左（在头部），而`RPUSH`命令添加新的
元素添加到列表中，位于右侧（在尾部）。最后
`LRANGE`命令从列表中提取元素范围：

    > rpush mylist A
    (integer) 1
    > rpush mylist B
    (integer) 2
    > lpush mylist first
    (integer) 3
    > lrange mylist 0 -1
    1) "first"
    2) "A"
    3) "B"

请注意，[兰奇](/commands/lrange)采用两个索引，第一个索引和最后一个索引
元素。两个索引都可以是负数，告诉Redis
从末尾开始计数：所以 -1 是最后一个元素，-2 是
列表的倒数第二个元素，依此类推。

如您所见`RPUSH`附加了列表右侧的元素，而
决赛`LPUSH`追加了左侧的元素。

这两个命令都是*可变参数命令*，这意味着您可以自由地推送
在单个调用中将多个元素放入列表中：

    > rpush mylist 1 2 3 4 5 "foo bar"
    (integer) 9
    > lrange mylist 0 -1
    1) "first"
    2) "A"
    3) "B"
    4) "1"
    5) "2"
    6) "3"
    7) "4"
    8) "5"
    9) "foo bar"

在 Redis 列表上定义的一个重要操作是*流行元素*.
弹出元素是从列表中检索元素的操作，
同时将其从列表中删除。您可以弹出元素
从左边和右边，类似于如何推动两侧的元素
的列表：

    > rpush mylist a b c
    (integer) 3
    > rpop mylist
    "c"
    > rpop mylist
    "b"
    > rpop mylist
    "a"

我们添加了三个元素并弹出了三个元素，所以在本文结束时
命令序列列表为空，并且没有更多元素
流行。如果我们尝试弹出另一个元素，这就是我们得到的结果：

    > rpop mylist
    (nil)

Redis 返回一个 NULL 值，以指示
列表。

### 列表的常见用例

列表对于许多任务都很有用，这是两个非常有代表性的用例
是以下各项：

*   请记住用户发布到社交网络中的最新更新。
*   进程之间的通信，使用使用者-生产者模式，其中生产者将项目推送到列表中，以及消费者（通常是*工人*） 使用这些项目并执行的操作。Redis 具有特殊的列表命令，使此用例更加可靠和高效。

例如，两个流行的Ruby库[重新调整](https://github.com/resque/resque)和
[sidekiq](https://github.com/mperham/sidekiq)在引擎盖下使用 Redis 列表，以便
实现后台作业。

流行的推特社交网络[获取最新推文](http://www.infoq.com/presentations/Real-Time-Delivery-Twitter)
由用户发布到 Redis 列表中。

要逐步描述常见用例，请想象一下您的主页显示最新的
照片发布在照片共享社交网络中，并且您希望加快访问速度。

*   每当用户发布新照片时，我们都会将其ID添加到列表中`LPUSH`.
*   当用户访问主页时，我们使用`LRANGE 0 9`为了获得最新的10个已发布的项目。

### 上限列表

在许多用例中，我们只想使用列表来存储*最新项目*,
无论它们是什么：社交网络更新，日志或其他任何东西。

Redis允许我们使用列表作为上限集合，只记住最新的
N 个项目并丢弃所有最旧的项目，使用`LTRIM`命令。

这`LTRIM`命令类似于`LRANGE`但**而不是显示
指定的元素范围**它将此范围设置为新的列表值。都
给定范围之外的元素将被删除。

一个例子可以更清楚地说明：

    > rpush mylist 1 2 3 4 5
    (integer) 5
    > ltrim mylist 0 2
    OK
    > lrange mylist 0 -1
    1) "1"
    2) "2"
    3) "3"

以上`LTRIM`命令告诉 Redis 仅从索引中获取列表元素
0到2，其他所有内容都将被丢弃。这允许一个非常简单，但
有用的模式：一起执行列表推送操作 + 列表修剪操作
为了添加新元素并丢弃超过限制的元素：

    LPUSH mylist <some element>
    LTRIM mylist 0 999

上述组合添加了一个新元素，并且仅采用 1000
列表中的最新元素。跟`LRANGE`您可以访问热门项目
无需记住非常旧的数据。

注意：虽然`LRANGE`从技术上讲是一个 O（N） 命令，可访问小范围
朝向列表的头部或尾部是一个恒定的时间操作。

## 对列表的阻止操作

列表具有使其适合实现队列的特殊功能，
并且通常作为过程间通信系统的构建块：
阻止操作。

想象一下，你想用一个进程将项目推送到列表中，并使用
一个不同的过程，以便实际与这些一起做某种工作
项目。这是通常的生产者/消费者设置，可以实现
以以下简单方式：

*   要将项目推送到列表中，生产者调用`LPUSH`.
*   要从列表中提取/处理项目，消费者调用`RPOP`.

但是，有时列表可能是空的，没有任何内容
要处理，所以`RPOP`只是返回 NULL。在这种情况下，消费者被迫等待
一段时间后重试`RPOP`.这称为*投票*，并且不是
在这种情况下，这是一个好主意，因为它有几个缺点：

1.  强制 Redis 和客户端处理无用的命令（当列表为空时，所有请求都不会完成任何实际工作，它们只会返回 NULL）。
2.  为物料的处理添加延迟，因为在工作线程收到 NULL 后，它会等待一段时间。为了减小延迟，我们可以在调用之间减少等待时间`RPOP`，具有放大问题1的效果，即对Redis的更多无用调用。

所以 Redis 实现了名为`BRPOP`和`BLPOP`哪些是版本
之`RPOP`和`LPOP`如果列表为空，则能够阻止：它们将返回到
仅当将新元素添加到列表中时，或者当用户指定
已达到超时。

这是一个示例`BRPOP`调用我们可以在工作线程中使用：

    > brpop tasks 5
    1) "tasks"
    2) "do_something"

这意味着：“等待列表中的元素`tasks`，但如果在 5 秒后返回
没有可用的元素”。

请注意，您可以使用 0 作为超时来永远等待元素，并且
还要指定多个列表，而不仅仅是一个列表，以便等待多个列表
同时列出，并在第一个列表收到
元素。

需要注意的几件事`BRPOP`:

1.  客户端以有序方式提供服务：阻止等待列表的第一个客户端，当元素被其他客户端推送时，首先提供服务，依此类推。
2.  返回值与`RPOP`：它是一个双元素数组，因为它还包括键的名称，因为`BRPOP`和`BLPOP`能够阻止等待来自多个列表的元素。
3.  如果达到超时，则返回 NULL。

关于列表和阻止操作，您应该了解更多内容。我们
建议您阅读以下内容：

*   可以使用以下命令构建更安全的队列或轮换队列`LMOVE`.
*   该命令还有一个阻塞变体，称为`BLMOVE`.

## 自动创建和删除密钥

到目前为止，在我们的示例中，我们从未在推送之前创建空列表
元素，或者在空列表中不再包含元素时将其删除。
Redis 负责在列表留空时删除密钥，或创建
如果键不存在并且我们正在尝试添加元素，则为空列表
例如，用`LPUSH`.

这并不特定于列表，它适用于所有 Redis 数据类型
由多个元素组成 - 流，集，排序集和哈希。

基本上，我们可以用三个规则来总结行为：

1.  将元素添加到聚合数据类型时，如果目标键不存在，则在添加元素之前会创建一个空的聚合数据类型。
2.  当我们从聚合数据类型中删除元素时，如果该值保持为空，则会自动销毁该键。流数据类型是此规则的唯一例外。
3.  调用只读命令，例如`LLEN`（返回列表的长度）或使用空键删除元素的 write 命令始终产生相同的结果，就好像该键包含该命令期望找到的类型为空聚合类型一样。

规则 1 的示例：

    > del mylist
    (integer) 1
    > lpush mylist 1 2 3
    (integer) 3

但是，如果密钥存在，则我们无法对错误的类型执行操作：

    > set foo bar
    OK
    > lpush foo 1 2 3
    (error) WRONGTYPE Operation against a key holding the wrong kind of value
    > type foo
    string

规则 2 的示例：

    > lpush mylist 1 2 3
    (integer) 3
    > exists mylist
    (integer) 1
    > lpop mylist
    "3"
    > lpop mylist
    "2"
    > lpop mylist
    "1"
    > exists mylist
    (integer) 0

弹出所有元素后，键不再存在。

规则 3 的示例：

    > del mylist
    (integer) 0
    > llen mylist
    (integer) 0
    > lpop mylist
    (nil)

<a name="hashes"></a>

## 散 列

Redis哈希值看起来与“哈希”完全相同，具有字段值对：

    > hset user:1000 username antirez birthyear 1977 verified 1
    (integer) 3
    > hget user:1000 username
    "antirez"
    > hget user:1000 birthyear
    "1977"
    > hgetall user:1000
    1) "username"
    2) "antirez"
    3) "birthyear"
    4) "1977"
    5) "verified"
    6) "1"

虽然哈希值便于表示*对象*，实际上您可以查看的字段数量
放在哈希中没有实际限制（可用内存除外），因此您可以使用
在应用程序中以许多不同的方式进行哈希处理。

命令`HSET`设置哈希的多个字段，而`HGET`检索
单个字段。`HMGET`类似于`HGET`但返回一个值数组：

    > hmget user:1000 username birthyear no-such-field
    1) "antirez"
    2) "1977"
    3) (nil)

有些命令能够对各个字段执行操作
以及，如`HINCRBY`:

    > hincrby user:1000 birthyear 10
    (integer) 1987
    > hincrby user:1000 birthyear 10
    (integer) 1997

您可以找到[文档中哈希命令的完整列表](https://redis.io/commands#hash).

值得注意的是，小哈希（即，一些具有小值的元素）是
在内存中以特殊方式编码，使它们非常具有内存效率。

<a name="sets"></a>

## 集

Redis 集是字符串的无序集合。这
`SADD`命令将新元素添加到集合中。这也是可能的
对集合执行许多其他操作，例如测试给定元素
已存在，执行交集、并集或差分
多个集合，依此类推。

    > sadd myset 1 2 3
    (integer) 3
    > smembers myset
    1. 3
    2. 1
    3. 2

在这里，我在我的集合中添加了三个元素，并告诉Redis返回所有
元素。如您所见，它们未排序 - Redis 可以自由返回
元素在每次调用时以任何顺序排列，因为没有与
用户关于元素排序。

Redis 具有用于测试成员资格的命令。例如，检查元素是否存在：

    > sismember myset 3
    (integer) 1
    > sismember myset 30
    (integer) 0

“3”是集合的成员，而“30”不是。

集合适用于表示对象之间的关系。
例如，我们可以很容易地使用集合来实现标签。

对这个问题进行建模的一种简单方法是为每个对象设置一个集合，
想要标记。该集包含与对象关联的标记的 ID。

一个例子是标记新闻文章。
如果文章 ID 1000 标记有标记 1、2、5 和 77，则一组
可以将这些标记 ID 与新闻项相关联：

    > sadd news:1000:tags 1 2 5 77
    (integer) 4

我们可能还希望具有反关系：列表
所有标记有给定标签的新闻：

    > sadd tag:1:news 1000
    (integer) 1
    > sadd tag:2:news 1000
    (integer) 1
    > sadd tag:5:news 1000
    (integer) 1
    > sadd tag:77:news 1000
    (integer) 1

获取给定对象的所有标签是微不足道的：

    > smembers news:1000:tags
    1. 5
    2. 1
    3. 77
    4. 2

注意：在示例中，我们假设您有另一个数据结构，例如
一个 Redis 哈希，它将标签 ID 映射到标签名称。

还有其他一些重要的操作仍然易于实现
使用正确的 Redis 命令。例如，我们可能想要一个列表
标签 1、2、10 和 27 在一起的对象。我们可以使用
这`SINTER`命令，它执行不同之间的交集
集。我们可以使用：

    > sinter tag:1:news tag:2:news tag:10:news tag:27:news
    ... results here ...

除了交叉路口，您还可以执行
并集、差值、提取随机元素等。

提取元素的命令称为`SPOP`，并且便于建模
某些问题。例如，为了实现基于Web的扑克游戏，
你可能想用一套来代表你的套牌。想象一下，我们使用一个字符
（C）lubs， （D）iamonds， （H）earts， （S）pades 的前缀：

    > sadd deck C1 C2 C3 C4 C5 C6 C7 C8 C9 C10 CJ CQ CK
      D1 D2 D3 D4 D5 D6 D7 D8 D9 D10 DJ DQ DK H1 H2 H3
      H4 H5 H6 H7 H8 H9 H10 HJ HQ HK S1 S2 S3 S4 S5 S6
      S7 S8 S9 S10 SJ SQ SK
    (integer) 52

现在我们想为每个玩家提供5张牌。这`SPOP`命令
删除一个随机元素，将其返回到客户端，因此它是
在这种情况下，完美的操作。

但是，如果我们直接针对我们的甲板调用它，则在下一个游戏中
游戏我们需要再次填充纸牌组，这可能不是
理想。所以首先，我们可以制作一个存储在`deck`钥匙
进入`game:1:deck`钥匙。

这是通过使用`SUNIONSTORE`，通常执行
多个集合之间的并集，并将结果存储到另一个集合中。
但是，由于单个集合的并集本身就是，因此我可以复制我的套牌
跟：

    > sunionstore game:1:deck deck
    (integer) 52

现在我准备为第一个玩家提供五张牌：

    > spop game:1:deck
    "C6"
    > spop game:1:deck
    "CQ"
    > spop game:1:deck
    "D1"
    > spop game:1:deck
    "CJ"
    > spop game:1:deck
    "SJ"

一对插孔，不是很好...

这是引入提供数字的set命令的好时机
集合内的元素。这通常称为*集合的基数*
在集合论的上下文中，因此 Redis 命令称为`SCARD`.

    > scard game:1:deck
    (integer) 47

数学运算：52 - 5 = 47。

当您只需要获取随机元素而不从
设置，有`SRANDMEMBER`命令适合于任务。它还具有以下特点：
能够同时返回重复和非重复元素。

<a name="sorted-sets"></a>

## 排序集

排序集是一种数据类型，类似于 Set 和
a 哈希。与集合一样，排序集合由唯一的、非重复的组成。
字符串元素，因此从某种意义上说，排序集也是一个集合。

但是，虽然集合内的元素没有排序，但
排序的集合与浮点值相关联，称为*分数*
（这就是为什么类型也类似于哈希，因为每个元素
映射到值）。

此外，排序集中的元素是*按顺序拍摄*（所以他们不是
根据请求排序，订单是用于
表示排序集）。它们根据以下规则进行排序：

*   如果 B 和 A 是两个分数不同的元素，则 A > B（如果 A.score > B.score）。
*   如果 B 和 A 具有完全相同的分数，则当 A 字符串在字典上大于 B 字符串时，A > B。B 和 A 字符串不能相等，因为排序集只有唯一的元素。

让我们从一个简单的例子开始，添加一些选定的黑客名字作为
排序的集合元素，其出生年份为“分数”。

    > zadd hackers 1940 "Alan Kay"
    (integer) 1
    > zadd hackers 1957 "Sophie Wilson"
    (integer) 1
    > zadd hackers 1953 "Richard Stallman"
    (integer) 1
    > zadd hackers 1949 "Anita Borg"
    (integer) 1
    > zadd hackers 1965 "Yukihiro Matsumoto"
    (integer) 1
    > zadd hackers 1914 "Hedy Lamarr"
    (integer) 1
    > zadd hackers 1916 "Claude Shannon"
    (integer) 1
    > zadd hackers 1969 "Linus Torvalds"
    (integer) 1
    > zadd hackers 1912 "Alan Turing"
    (integer) 1

如您所见`ZADD`类似于`SADD`，但需要一个额外的参数
（放置在要添加的元素之前）这是分数。
`ZADD`也是可变参数，因此您可以自由指定多个分数值
配对，即使在上面的示例中未使用。

使用排序集，返回按其排序的黑客列表是微不足道的
出生年份，因为实际上*它们已被排序*.

实现说明：排序集通过
双端口数据结构，包含跳过列表和哈希表，因此
每次我们添加元素时，Redis都会执行O（log（N））操作。那是
很好，但是当我们要求排序元素时，Redis不必在
所有，它已经全部排序：

    > zrange hackers 0 -1
    1) "Alan Turing"
    2) "Hedy Lamarr"
    3) "Claude Shannon"
    4) "Alan Kay"
    5) "Anita Borg"
    6) "Richard Stallman"
    7) "Sophie Wilson"
    8) "Yukihiro Matsumoto"
    9) "Linus Torvalds"

注意：0 和 -1 表示从元素索引 0 到最后一个元素 （-1 工作
在这里，就像在以下情况下一样`LRANGE`命令）。

如果我想以相反的方式订购它们，从最年轻到最年长，该怎么办？
用[兹雷夫兰奇](/commands/zrevrange)而不是[兹兰奇](/commands/zrange):

    > zrevrange hackers 0 -1
    1) "Linus Torvalds"
    2) "Yukihiro Matsumoto"
    3) "Sophie Wilson"
    4) "Richard Stallman"
    5) "Anita Borg"
    6) "Alan Kay"
    7) "Claude Shannon"
    8) "Hedy Lamarr"
    9) "Alan Turing"

也可以使用`WITHSCORES`论点：

    > zrange hackers 0 -1 withscores
    1) "Alan Turing"
    2) "1912"
    3) "Hedy Lamarr"
    4) "1914"
    5) "Claude Shannon"
    6) "1916"
    7) "Alan Kay"
    8) "1940"
    9) "Anita Borg"
    10) "1949"
    11) "Richard Stallman"
    12) "1953"
    13) "Sophie Wilson"
    14) "1957"
    15) "Yukihiro Matsumoto"
    16) "1965"
    17) "Linus Torvalds"
    18) "1969"

### 在范围上运行

排序集比这更强大。它们可以在量程上运行。
让我们获取所有在1950年之前出生的人（包括1950年）。我们
使用`ZRANGEBYSCORE`命令来做到这一点：

    > zrangebyscore hackers -inf 1950
    1) "Alan Turing"
    2) "Hedy Lamarr"
    3) "Claude Shannon"
    4) "Alan Kay"
    5) "Anita Borg"

我们要求 Redis 返回所有得分介于负值之间的元素
无穷大和 1950 年（两个极端都包括在内）。

还可以删除元素范围。让我们全部删除
1940年至1960年间出生的黑客从排序的集合中：

    > zremrangebyscore hackers 1940 1960
    (integer) 4

`ZREMRANGEBYSCORE`也许不是最好的命令名称，
但它可能非常有用，并返回已删除元素的数量。

为排序集元素定义的另一个非常有用的操作
是获取排名操作。可以问什么是
元素在有序元素集中的位置。

    > zrank hackers "Anita Borg"
    (integer) 4

这`ZREVRANK`命令也可用，以便获得排名，考虑
元素以降序方式排序。

### 词典编纂分数

在最新版本的 Redis 2.8 中，引入了一项新功能，该功能允许
从字典上获取范围，假设排序集中的元素全部
插入相同的分数（元素与C进行比较）
`memcmp`函数，因此可以保证没有排序规则，并且每个
Redis 实例将使用相同的输出进行回复）。

使用词典范围进行操作的主要命令是`ZRANGEBYLEX`,
`ZREVRANGEBYLEX`,`ZREMRANGEBYLEX`和`ZLEXCOUNT`.

例如，让我们再次添加我们的着名黑客列表，但这次
对所有元素使用零分数：

    > zadd hackers 0 "Alan Kay" 0 "Sophie Wilson" 0 "Richard Stallman" 0
      "Anita Borg" 0 "Yukihiro Matsumoto" 0 "Hedy Lamarr" 0 "Claude Shannon"
      0 "Linus Torvalds" 0 "Alan Turing"

由于排序集排序规则，它们已经过排序
词典编纂：

    > zrange hackers 0 -1
    1) "Alan Kay"
    2) "Alan Turing"
    3) "Anita Borg"
    4) "Claude Shannon"
    5) "Hedy Lamarr"
    6) "Linus Torvalds"
    7) "Richard Stallman"
    8) "Sophie Wilson"
    9) "Yukihiro Matsumoto"

用`ZRANGEBYLEX`我们可以要求词典编纂范围：

    > zrangebylex hackers [B [P
    1) "Claude Shannon"
    2) "Hedy Lamarr"
    3) "Linus Torvalds"

范围可以是包含的，也可以是独占的（取决于第一个字符），
字符串无限和负无限分别指定为
这`+`和`-`字符串。有关详细信息，请参阅文档。

此功能很重要，因为它允许我们使用排序集作为泛型
指数。例如，如果要按 128 位无符号对元素编制索引
整数参数，您需要做的就是将元素添加到排序中
使用相同的分数（例如 0）设置，但使用 16 字节前缀
包括**大端序中的 128 位数字**.由于数字在大
字节序，当按字典顺序排序时（以原始字节顺序）实际上是
在数字上也是有序的，您可以要求在128位空间中的范围，
并获取元素的值，放弃前缀。

如果您想在更严肃的演示中查看该功能，
检查[Redis 自动完成演示](http://autocomplete.redis.io).

## 更新分数：排行榜

在切换到下一个主题之前，只是关于排序集的最后一点说明。
排序集的分数可以随时更新。只需致电`ZADD`对
已包含在排序集中的元素将更新其分数
（和位置）具有 O（log（N）） 时间复杂度。 因此，排序集是合适的
当有大量的更新时。

由于这个特征，一个常见的用例是排行榜。
典型的应用程序是Facebook游戏，您可以在其中结合以下功能：
按高分对用户进行排序，加上获取排名操作，按顺序排序
以显示前 N 个用户，以及用户在排行榜中的排名（例如，“您是
\#4932 最佳分在这里”）。

<a name="bitmaps"></a>

## 位图

位图不是实际的数据类型，而是一组面向位的操作
在字符串类型上定义。由于字符串是二进制安全 blob 及其
最大长度为512 MB，它们适合设置多达2^ 32个不同的
位。

位操作分为两组：恒定时间单比特
操作，例如将位设置为 1 或 0，或获取其值，以及
对位组的操作，例如计算集合数
给定位范围内的位（例如，人口计数）。

位图的最大优点之一是它们通常提供
在存储信息时节省极大的空间。例如，在系统中
如果不同的用户由增量用户 ID 表示，则有可能
记住单个位信息（例如，知道是否
用户希望收到40亿用户的新闻通讯，仅使用512 MB的内存。

位是使用`SETBIT`和`GETBIT`命令：

    > setbit key 10 1
    (integer) 1
    > getbit key 10
    (integer) 1
    > getbit key 11
    (integer) 0

这`SETBIT`命令将位数作为其第一个参数，并将其第二个参数
参数：将位设置为的值，即 1 或 0。命令
如果寻址位在
当前字符串长度。

`GETBIT`只是返回指定索引处的位的值。
超出范围的位（寻址超出字符串长度的位）
存储在目标键中）始终被视为零。

有三个命令对位组进行操作：

1.  `BITOP`在不同字符串之间执行按位运算。提供的操作是 AND、OR、XOR 和 NOT。
2.  `BITCOUNT`执行总体计数，报告设置为 1 的位数。
3.  `BITPOS`查找具有指定值 0 或 1 的第一个位。

双`BITPOS`和`BITCOUNT`能够在
字符串，而不是为字符串的整个长度运行。以下
是一个微不足道的例子`BITCOUNT`叫：

    > setbit key 0 1
    (integer) 0
    > setbit key 100 1
    (integer) 0
    > bitcount key
    (integer) 2

位图的常见用例包括：

*   各种实时分析。
*   存储与对象 ID 关联的高效但高性能的布尔信息。

例如，想象一下，您想知道最长的每日访问记录
您的网站用户。您从零开始计算天数，即
你把你的网站公开的那一天，并设置了一点`SETBIT`每次
用户访问网站。作为位索引，您只需采用当前的unix
time，减去初始偏移量，然后除以一天中的秒数
（通常为 3600\*24）。

这样，对于每个用户，您都有一个包含访问的小字符串
每天的信息。跟`BITCOUNT`可以很容易地得到
给定用户访问网站的天数，而
一些`BITPOS`调用，或者只是获取和分析位图客户端，
可以很容易地计算出最长的条纹。

位图很容易拆分为多个键，例如
为了分片数据集，因为一般来说，它更适合
避免使用大键。跨不同键拆分位图
而不是将所有位设置为一个键，一个微不足道的策略只是
为每个密钥存储 M 位，并获取密钥名称`bit-number/M`和
密钥内要寻址的第 N 位`bit-number MOD M`.

<a name="hyperloglogs"></a>

## 超级日志

HyperLogLog是用于计数的概率数据结构
独特的事物（从技术上讲，这是指估计基数
的集合）。通常，计算唯一项目需要使用内存量
与您要计数的项目数量成比例，因为您需要
记住你过去已经看到的元素，以避免
多次计数。但是，有一组算法可以进行交易
精度的内存：以标准误差的估计度量值结尾，
在Redis实现的情况下，小于1%。 这
这种算法的神奇之处在于，您不再需要使用大量内存
与计数的项目数成比例，并且可以使用
恒定的内存量！在最坏的情况下为12k字节，或者如果您的
HyperLogLog（从现在开始我们只称它们为HLL）已经看到了很少的元素。

Redis中的HLL虽然在技术上具有不同的数据结构，但被编码
作为 Redis 字符串，因此您可以调用`GET`序列化 HLL，以及`SET`
以将其反序列化回服务器。

从概念上讲，HLL API就像使用集合来执行相同的任务。你会
`SADD`每个观察到的元素都成一个集合，并且会使用`SCARD`以检查
集合中元素的数量，这些元素是唯一的，因为`SADD`不会
重新添加现有元素。

虽然你真的没有*添加项目*进入HLL，因为数据结构
只包含不包含实际元素的状态，API 是
相同：

*   每次看到新元素时，您都可以将其添加到计数中`PFADD`.
*   每次要检索唯一元素的当前近似值时*添加*跟`PFADD`到目前为止，您使用的`PFCOUNT`.

          > pfadd hll a b c d
          (integer) 1
          > pfcount hll
          (integer) 4

此数据结构的用例示例是计算唯一查询
由用户每天在搜索表单中执行。

Redis 还能够执行 HLL 的并集，请检查
[完整文档](/commands#hyperloglog)了解更多信息。

## 其他值得注意的功能

Redis API 中还有其他一些重要的事情无法探索
在本文档的上下文中，但值得您注意：

*   有可能[以增量方式迭代大型集合的密钥空间](/commands/scan).
*   可以运行[Lua 脚本服务器端](/commands/eval)以改善延迟和带宽。
*   Redis也是一个[发布-订阅服务器](/topics/pubsub).

## 了解更多信息

本教程绝不完整，仅介绍了 API 的基础知识。
阅读[命令引用](/commands)发现更多。

感谢您的阅读，并享受Redis黑客的乐趣！
