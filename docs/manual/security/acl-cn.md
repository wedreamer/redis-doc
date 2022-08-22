---
title: "ACL"
linkTitle: "ACL"
weight: 1
description: Redis access control list
aliases:
    - /topics/acl
---

Redis ACL 是访问控制列表的缩写，它允许某些连接在可执行的命令和可访问的键方面受到限制。它的工作方式是，连接后，客户端需要提供用户名和有效密码才能进行身份验证。如果身份验证成功，则连接与给定用户和用户的限制相关联。可以配置 Redis，以便新连接已经通过“默认”用户身份验证（这是默认配置）。配置默认用户的副作用是，能够仅向未显式验证的连接提供特定的功能子集。

在默认配置中，Redis 6（第一个具有 ACL 的版本）的工作方式与旧版本的 Redis 完全相同。每个新连接都能够调用每个可能的命令并访问每个键，因此 ACL 功能向后兼容旧客户端和应用程序。此外，使用 **requirepass** 配置指令配置密码的旧方法仍然可以按预期工作。但是，它现在为默认用户设置密码。

Redis `AUTH` 命令在 Redis 6 中进行了扩展，因此现在可以以两个参数的形式使用它：

    AUTH <username> <password>

这是旧形式的示例：

    AUTH <password>

发生的情况是用于身份验证的用户名是“默认”，因此仅指定密码意味着我们要针对默认用户进行身份验证。这提供了向后兼容性。

## 当 ACL 有用时

在使用 ACL 之前，您可能想问自己，通过实施这一层保护，您想要实现的目标是什么。通常，ACL 可以很好地服务于两个主要目标。

1.  您希望通过限制对命令和密钥的访问来提高安全性, 以便不受信任的客户端没有访问权限, 而受信任的客户端仅具有对数据库的最低访问级别, 以便执行所需的工作。例如, 某些客户端可能只能执行只读命令。
2.  您希望提高操作安全性, 以便不允许访问 Redis 的进程或人员因软件错误或手动错误而损坏数据或配置。例如, 从 Redis 获取延迟作业的辅助角色没有理由能够调用`FLUSHALL`命令。

ACL 的另一个典型用法与托管 Redis 实例有关。 Redis 通常由为他们拥有的其他内部客户处理 Redis 基础架构的内部公司团队作为托管服务提供，或者由云提供商在软件即服务设置中提供。在这两种设置中，我们都希望确保为客户排除配置命令。

## 使用 ACL 命令配置 ACL

ACL 是使用 DSL（领域特定语言）定义的，该语言描述了允许给定用户执行的操作。这样的规则总是从第一个到最后一个，从左到右执行，因为有时规则的顺序对于理解用户真正能够做什么很重要。

默认情况下，定义了一个用户，称为 *default*。我们可以使用 `ACL LIST` 命令来检查当前活动的 ACL 并验证新启动的、默认配置的 Redis 实例的配置是什么：

    > ACL LIST
    1) "user default on nopass ~* &* +@all"

上面的命令以与 Redis 配置文件中使用的相同格式报告用户列表，方法是将当前为用户设置的 ACL 转换回他们的描述。

每行的前两个单词是“user”，后跟用户名。接下来的词是描述不同事物的 ACL 规则。我们将详细展示规则是如何工作的，但现在只需将默认用户配置为活动 (on)、不需要密码 (nopass)、访问所有可能的密钥 (`~*` ) 和 Pub/Sub 频道 (`&*`)，并且能够调用所有可能的命令 (`+@all`).

此外，在默认用户的特殊情况下，拥有 *nopass* 规则意味着新连接会自动与默认用户进行身份验证，而无需任何显式的 `AUTH` 调用。

## ACL 规则

以下是有效 ACL 规则的列表。某些规则只是用于激活或删除标志或对用户 ACL 执行给定更改的单个词。其他规则是与命令或类别名称、键模式等连接的字符前缀。

启用和禁止用户：

*   `on`：启用用户：可以以此用户身份进行身份验证。
*   `off`：禁止用户：无法再对此用户进行身份验证;但是, 以前经过身份验证的连接仍将有效。请注意, 如果默认用户被标记为*关闭*, 新连接将以未通过身份验证的方式启动, 并要求用户发送`AUTH`或`HELLO`使用AUTH选项, 以便以某种方式进行身份验证, 而不管默认用户配置如何。

允许和禁止命令：

*   `+<command>`：将命令添加到用户可以调用的命令列表中。可与`|`用于允许子命令 (例如“+config|get”) 。
*   `-<command>`：将命令删除到用户可以调用的命令列表中。从 Redis 7.0 开始, 它可以与`|`用于阻塞子命令 (例如“-config|set”) 。
*   `+@<category>`：添加此类类别中的所有命令以供用户调用, 有效类别如@admin, @set, @sortedset, ...依此类推, 通过调用`ACL CAT`命令。特殊类别@all是指所有命令, 包括服务器中当前存在的命令, 以及将来将通过模块加载的命令。
*   `-@<category>`： 赞`+@<category>`但从客户端可以调用的命令列表中删除命令。
*   `+<command>|first-arg`：允许以其他方式禁用的命令的特定第一个参数。它仅在没有子命令的命令上受支持, 并且不允许作为负形式 (如 -SELECT|1) , 仅以“+”开头的加法。此功能已弃用, 将来可能会被删除。
*   `allcommands`：+@all的别名。请注意, 这意味着能够执行通过模块系统加载的所有未来命令。
*   `nocommands`：-@all的别名。

允许和禁止某些密钥和密钥权限：

*   `~<pattern>`：添加可作为命令的一部分提及的键模式。例如`~*`允许所有键。该模式是球形样式的模式, 如`KEYS`.可以指定多个模式。
*   `%R~<pattern>`： (在 Redis 7.0 及更高版本中可用) 添加指定的读取键模式。这类似于常规密钥模式, 但仅授予读取与给定模式匹配的密钥的权限。看[密钥权限](#key-permissions)了解更多信息。
*   `%W~<pattern>`： (在 Redis 7.0 及更高版本中可用) 添加指定的写入密钥模式。这类似于常规密钥模式, 但仅授予写入与给定模式匹配的密钥的权限。看[密钥权限](#key-permissions)了解更多信息。
*   `%RW~<pattern>`： (在 Redis 7.0 及更高版本中可用) 别名`~<pattern>`.
*   `allkeys`：别名`~*`.
*   `resetkeys`：刷新允许的键模式列表。例如, ACL`~foo:* ~bar:* resetkeys ~objects:*`, 将仅允许客户端访问与模式匹配的密钥`objects:*`.

允许和禁止发布/订阅频道：

*   `&<pattern>`： (在 Redis 6.2 及更高版本中可用) 添加用户可访问的 Pub/Sub 频道的 glob 样式模式。可以指定多个通道模式。请注意, 模式匹配仅对`PUBLISH`和`SUBSCRIBE`而`PSUBSCRIBE`需要其通道模式与允许用户的通道模式之间的文字匹配。
*   `allchannels`：别名`&*`允许用户访问所有发布/订阅频道。
*   `resetchannels`：刷新允许的通道模式列表, 如果用户的 Pub/Sub 客户端不再能够访问其各自的通道和/或通道模式, 请断开这些客户端的连接。

为用户配置有效密码：

*   `><password>`：将此密码添加到用户的有效密码列表中。例如`>mypass`将“mypass”添加到有效密码列表中。 此指令清除了*诺通*标志 (见后文) 。每个用户都可以拥有任意数量的密码。
*   `<<password>`：从有效密码列表中删除此密码。在您尝试删除的密码实际上未设置的情况下发出错误。
*   `#<hash>`：将此 SHA-256 哈希值添加到用户的有效密码列表中。此哈希值将与为 ACL 用户输入的密码的哈希值进行比较。这允许用户将哈希存储在`acl.conf`文件, 而不是存储明文密码。仅接受 SHA-256 哈希值, 因为密码哈希必须为 64 个字符, 并且仅包含小写十六进制字符。
*   `!<hash>`：从有效密码列表中删除此哈希值。当您不知道哈希值指定的密码但想要从用户中删除密码时, 这很有用。
*   `nopass`：用户的所有设置密码都将被删除, 并且用户被标记为不需要密码：这意味着每个密码都将对此用户起作用。如果此指令用于默认用户, 则每个新连接将立即使用默认用户进行身份验证, 而无需任何显式 AUTH 命令。请注意, *重置通行证*指令将清除此条件。
*   `resetpass`：刷新允许的密码列表并删除*诺通*地位。后*重置通行证*, 则用户没有关联的密码, 并且无法在不添加某些密码 (或将其设置为) 的情况下进行身份验证*诺通*稍后) 。

*注意：如果用户未被标记为 nopass 并且没有有效密码列表, 则该用户实际上无法使用, 因为无法以该用户身份登录。*

为用户配置选择器：

*   `(<rule list>)`： (在 Redis 7.0 及更高版本中可用) 创建新的选择器来匹配规则。选择器在用户权限之后进行评估, 并根据定义的顺序进行评估。如果命令与用户权限或任何选择器匹配, 则允许该命令。看[选择](#selectors)了解更多信息。
*   `clearselectors`： (在 Redis 7.0 及更高版本中可用) 删除附加到用户的所有选择器。

重置用户：

*   `reset`执行以下操作：重置通道、重置密钥、重置通道、关闭、-@all。用户将返回到创建后立即恢复的相同状态。

## 使用 ACL SETUSER 命令创建和编辑用户 ACL

可以通过两种主要方式创建和修改用户：

1.  使用 ACL 命令及其`ACL SETUSER`子命令。
2.  修改可以定义用户的服务器配置, 然后重新启动服务器。使用*外部 ACL 文件*, 只需调用`ACL LOAD`.

在本节中, 我们将学习如何使用`ACL`命令。
有了这些知识，通过配置文件做同样的事情就很简单了。在配置中定义用户值得单独讨论，稍后将单独讨论。

首先, 尝试最简单的`ACL SETUSER`命令调用：

    > ACL SETUSER alice
    OK

`ACL SETUSER` 命令接受用户名和 ACL 规则列表以应用于用户。但是，上面的示例根本没有指定任何规则。
如果用户不存在，这将只创建用户，使用新用户的默认值。如果用户已经存在，上面的命令什么都不做。

检查默认用户状态：

    > ACL LIST
    1) "user alice off &* -@all"
    2) "user default on nopass ~* ~& +@all"

新用户“alice”是：

*   在关闭状态下, 因此`AUTH`将不适用于用户“爱丽丝”。
*   用户也没有设置密码。
*   无法访问任何命令。请注意, 默认情况下创建用户时无法访问任何命令, 因此`-@all`在上面的输出中可以省略;然而`ACL LIST`试图是显式的而不是隐式的。
*   没有用户可以访问的关键模式。
*   用户可以访问所有发布/订阅频道。

默认情况下, 新用户是使用限制性权限创建的。从 Redis 6.2 开始, ACL 还提供发布/订阅通道访问管理。为了在升级到 Redis 6.2 时确保与 6.0 版的向后兼容性, 默认情况下, 新用户将被授予“所有通道”权限。默认值可设置为`resetchannels`通过`acl-pubsub-default`配置指令。

从 7.0 开始, `acl-pubsub-default`值设置为`resetchannels`以默认限制通道访问, 以提供更好的安全性。
默认值可设置为`allchannels`通过`acl-pubsub-default`配置指令, 以便与以前的版本兼容。

这样的用户完全没用。让我们尝试定义用户，使其处于活动状态，有密码，并且可以仅使用 `GET` 命令访问以字符串“cached:”开头的键名。

    > ACL SETUSER alice on >p1pp0 ~cached:* +get
    OK

现在用户可以做一些事情, 但会拒绝做其他事情：

    > AUTH alice p1pp0
    OK
    > GET foo
    (error) NOPERM this user has no permissions to access one of the keys used as arguments
    > GET cached:1234
    (nil)
    > SET cached:1234 zap
    (error) NOPERM this user has no permissions to run the 'set' command

事情按预期工作。为了检查用户 alice 的配置（记住用户名是区分大小写的），可以使用 `ACL LIST` 的替代品，它被设计为更适合计算机阅读，而 `ACL GETUSER` 是更具人类可读性。

    > ACL GETUSER alice
    1) "flags"
    2) 1) "on"
       2) "allchannels"
    3) "passwords"
    4) 1) "2d9c75..."
    5) "commands"
    6) "-@all +get"
    7) "keys"
    8) "~cached:*"
    9) "channels"
    10) "&*"
    11) "selectors"
    12) 1) 1) "commands"
           2) "-@all +set"
           3) "keys"
           4) "~*"
           5) "channels"
           6) "&*"

这`ACL GETUSER`返回一个字段值数组, 该数组以更易于解析的术语描述用户。输出包括标志集、密钥模式列表、密码等。如果我们使用 RESP3, 则输出可能更具可读性, 以便将其作为映射回复返回：

    > ACL GETUSER alice
    1# "flags" => 1~ "on"
       2~ "allchannels"
    2# "passwords" => 1) "2d9c75273d72b32df726fb545c8a4edc719f0a95a6fd993950b10c474ad9c927"
    3# "commands" => "-@all +get"
    4# "keys" => "~cached:*"
    5# "channels" => "&*"
    6# "selectors" => 1) 1# "commands" => "-@all +set"
        2# "keys" => "~*"
        3# "channels" => "&*"

*注意：从现在开始, 我们将继续使用 Redis 默认协议版本 2*

使用另一个`ACL SETUSER`命令 (来自其他用户, 因为 alice 无法运行`ACL`命令) , 我们可以向用户添加多个模式：

    > ACL SETUSER alice ~objects:* ~items:* ~public:*
    OK
    > ACL LIST
    1) "user alice on >2d9c75... ~cached:* ~objects:* ~items:* ~public:* &* -@all +get"
    2) "user default on nopass ~* &* +@all"

内存中的用户表示形式现在符合我们的预期。

## 对 ACL SETUSER 的多次调用

了解多次调用 `ACL SETUSER` 时会发生什么非常重要。重要的是要知道每个 `ACL SETUSER` 调用都不会重置用户，而只会将 ACL 规则应用于现有用户。
仅当用户之前不知道时才会重置用户。在这种情况下，将使用零 ACL 创建一个全新的用户。用户不能做任何事情、被禁止、没有密码等等。这是安全的最佳默认值。

但是, 以后的调用只会以增量方式修改用户。例如以下序列：

    > ACL SETUSER myuser +set
    OK
    > ACL SETUSER myuser +get
    OK

将导致用户能够同时调用`GET`和`SET`:

    > ACL LIST
    1) "user default on nopass ~* &* +@all"
    2) "user myuser off &* -@all +set +get"

## 命令类别

通过一个接一个地指定所有命令来设置用户 ACL 真的很烦人，所以我们做这样的事情：

    > ACL SETUSER antirez on +@all -@dangerous >42a979... ~*

通过说 +@all 和 -@dangerous，我们包含了所有命令，后来删除了 Redis 命令表中所有标记为危险的命令。
请注意，命令类别**从不包括模块命令**，+@all 除外。如果你说 +@all，所有的命令都可以由用户执行，甚至未来通过模块系统加载的命令。但是，如果您使用 ACL 规则 +@read 或任何其他规则，则始终排除模块命令。这非常重要，因为您应该只信任 Redis 内部命令表。模块可能会暴露危险的东西，并且在 ACL 只是附加的情况下，即以 `+@all -...` 的形式。
你应该绝对确定你永远不会包含你不想要的东西。

以下是命令类别及其含义的列表：

*   **管理**- 管理命令。普通应用程序将永远不需要使用
    这些。包括`REPLICAOF`,`CONFIG`,`DEBUG`,`SAVE`,`MONITOR`,`ACL`,`SHUTDOWN`等。
*   **位图**- 数据类型：位图相关。
*   **阻塞**- 可能阻止连接, 直到被另一个释放
    命令。
*   **连接**- 影响连接或其他连接的命令。
    这包括`AUTH`,`SELECT`,`COMMAND`,`CLIENT`,`ECHO`,`PING`等。
*   **危险**- 潜在危险的命令 (每个命令都应谨慎考虑
    各种原因) 。这包括`FLUSHALL`,`MIGRATE`,`RESTORE`,`SORT`,`KEYS`,
    `CLIENT`,`DEBUG`,`INFO`,`CONFIG`,`SAVE`,`REPLICAOF`等。
*   **地理**- 数据类型：地理空间索引相关。
*   **散 列**- 数据类型：哈希相关。
*   **超日志**- 数据类型：与超日志相关。
*   **快**- 快速 O (1)  命令。可以循环参数的数量, 但不是
    键中的元素数。
*   **键空间**- 写入或读取密钥、数据库或其元数据
    以与类型无关的方式。包括`DEL`,`RESTORE`,`DUMP`,`RENAME`,`EXISTS`,`DBSIZE`,
    `KEYS`,`EXPIRE`,`TTL`,`FLUSHALL`等。可能修改键空间的命令, 
    键或元数据也将具有`write`类别。仅读取的命令
    键空间、键或元数据将具有`read`类别。
*   **列表**- 数据类型：相关列表。
*   **酒吧酒吧**- 与 PubSub 相关的命令。
*   **读**- 从键 (值或元数据) 读取。请注意, 不带的命令
    与密钥交互, 不会有`read`或`write`.
*   **脚本**- 脚本相关。
*   **设置**- 数据类型：集合相关。
*   **排序集**- 数据类型：排序集相关。
*   **慢**- 所有不是的命令`fast`.
*   **流**- 数据类型：流相关。
*   **字符串**- 数据类型：字符串相关。
*   **交易**-`WATCH`/`MULTI`/`EXEC`相关命令。
*   **写**- 写入键 (值或元数据) 。

Redis还可以使用Redis向您显示所有类别的列表以及每个类别包含的确切命令`ACL`命令的`CAT`子命令。它可以以两种形式使用：

    ACL CAT -- Will just list all the categories available
    ACL CAT <category-name> -- Will list all the commands inside the category

例子：

     > ACL CAT
     1) "keyspace"
     2) "read"
     3) "write"
     4) "set"
     5) "sortedset"
     6) "list"
     7) "hash"
     8) "string"
     9) "bitmap"
    10) "hyperloglog"
    11) "geo"
    12) "stream"
    13) "pubsub"
    14) "admin"
    15) "fast"
    16) "slow"
    17) "blocking"
    18) "dangerous"
    19) "connection"
    20) "transaction"
    21) "scripting"

如您所见，到目前为止，共有 21 个不同的类别。现在让我们检查一下哪个命令属于 *geo* 类别：

    > ACL CAT geo
    1) "geohash"
    2) "georadius_ro"
    3) "georadiusbymember"
    4) "geopos"
    5) "geoadd"
    6) "georadiusbymember_ro"
    7) "geodist"
    8) "georadius"

请注意，命令可能属于多个类别。例如，像 `+@geo -@read` 这样的 ACL 规则将导致某些地理命令被排除，因为它们是只读命令。

## 允许/禁止子命令

从 Redis 7.0 开始，可以像其他命令一样允许/阻止子命令（通过在命令和子命令之间使用分隔符 `|`，例如：`+config|get` 或 `-config|set`）

除 DEBUG 之外的所有命令都是如此。为了允许/阻止特定的 DEBUG 子命令，请参阅下一节。

## 允许被阻止命令的第一个参数

**注意：自 Redis 7.0 起, 此功能已弃用, 将来可能会被删除。**

有时, 排除或包含命令或子命令作为一个整体的能力是不够的。
许多部署可能不乐意为任何数据库提供执行“SELECT”的能力，但可能仍希望能够运行“SELECT 0”.

在这种情况下, 我们可以通过以下方式更改用户的 ACL：

    ACL SETUSER myuser -select +select|0

首先，删除 `SELECT` 命令，然后添加允许的 first-arg。请注意，**不可能做相反的事情**，因为 first-args 只能添加，不能排除。指定对某些用户有效的所有 first-args 更安全，因为将来可能会添加新的 first-args。

另一个例子：

    ACL SETUSER myuser -debug +debug|digest

请注意，第一个参数匹配可能会增加一些性能损失；然而，即使使用综合基准也很难衡量。仅在调用此类命令时支付额外的 CPU 成本，而不是在调用其他命令时支付。

可以使用此机制以允许在 7.0 之前的 Redis 版本中使用子命令（请参阅上一节）。

## +@all VS -@all

在上一节中，观察到如何基于添加/删除单个命令来定义命令 ACL。

## 选择

从 Redis 7.0 开始, Redis 支持添加多组规则, 这些规则是相互独立评估的。
这些辅助权限集称为选择器, 并通过在括号内包装一组规则来添加。
为了执行命令, root 权限 (在括号外定义的规则) 或任何选择器 (在括号内定义的规则) 必须与给定的命令匹配。
在内部, 首先检查根权限, 然后按添加顺序检查选择器。

例如, 考虑具有 ACL 规则的用户`+GET ~key1 (+SET ~key2)`.
此用户能够执行`GET key1`和`SET key2 hello`, 但不是`GET key2`或`SET key1 world`.

与用户的 root 权限不同, 选择器在添加后无法修改。
相反, 可以选择器可以通过`clearselectors`关键字, 用于删除所有添加的选择器。
请注意, `clearselectors`不会删除根权限。

## 密钥权限

从 Redis 7.0 开始, 键模式还可用于定义命令如何触摸键。
这是通过定义密钥权限的规则实现的。
关键权限规则采用以下形式`%(<permission>)~<pattern>`.
权限定义为映射到以下关键权限的单个字符：

*   W (写入) ：密钥中存储的数据可能会被更新或删除。
*   R (读取) ：处理、复制或返回用户从密钥提供的数据。请注意, 这不包括元数据, 例如大小信息 (示例`STRLEN`) 、类型信息 (示例`TYPE`) 或有关集合中是否存在值的信息 (示例`SISMEMBER`).

可以通过指定多个字符来组合权限。
将权限指定为“RW”被视为完全访问权限, 类似于仅传入`~<pattern>`.

有关具体示例, 请考虑具有 ACL 规则的用户`+@all ~app1:* (+@readonly ~app2:*)`.
此用户对`app1:*`和只读访问`app2:*`.
但是, 某些命令支持从一个键读取数据, 执行一些转换, 然后将其存储到另一个键中。
一个这样的命令是`COPY`命令, 将数据从源密钥复制到目标密钥。
示例 ACL 规则集无法处理从`app2:user`到`app1:user`, 因为 root 权限或选择器都不完全匹配该命令。
但是, 使用键选择器, 您可以定义一组可以处理此请求的 ACL 规则`+@all ~app1:* %R~app2:*`.
第一种模式能够匹配`app1:user`并且第二种模式能够匹配`app2:user`.

命令需要哪种类型的权限通过以下方式记录[主要规格](/topics/key-specs#logical-operation-flags).
权限类型基于键逻辑操作标志。
插入、更新和删除标志映射到写入密钥权限。
访问标志映射到读取密钥权限。
如果密钥没有逻辑操作标志, 例如`EXISTS`, 用户仍然需要密钥读取或密钥写入权限来执行命令。

注意：在评估执行命令是否需要读取权限时, 将忽略访问用户数据的侧通道。
这意味着某些返回有关已修改密钥的元数据的写入命令只需要对密钥的写入权限即可执行：
例如, 请考虑以下两个命令：

*   `LPUSH key1 data`：修改“key1”但只返回有关它的元数据, 推送后列表的大小, 因此该命令只需要对“key1”的写入权限即可执行。
*   `LPOP key2`：修改“key2”, 但也从列表中最左边的项目返回数据, 因此该命令需要对“key2”的读取和写入权限才能执行。

如果应用程序需要确保没有从密钥 (包括侧通道) 访问数据, 则建议不要提供对密钥的任何访问。

## 密码如何在内部存储

Redis internally stores passwords hashed with SHA256. If you set a password and check the output of `ACL LIST` or `ACL GETUSER`, you'll see a long hex string that looks pseudo random. Here is an example, because in the previous examples, for the sake of brevity, the long hex string was trimmed：

    > ACL GETUSER default
    1) "flags"
    2) 1) "on"
       2) "allkeys"
       3) "allcommands"
       4) "allchannels"
    3) "passwords"
    4) 1) "2d9c75273d72b32df726fb545c8a4edc719f0a95a6fd993950b10c474ad9c927"
    5) "commands"
    6) "+@all"
    7) "keys"
    8) "~*"
    9) "channels"
    10) "&*"
    11) "selectors"
    12) (empty array)

此外，从 Redis 6 开始，旧命令 `CONFIG GET requirepass` 将不再返回明文密码，而是返回哈希密码。

使用 SHA256 可以避免以明文形式存储密码，同时仍然允许非常快速的“AUTH”命令，这是 Redis 的一个非常重要的特性，并且符合客户对 Redis 的期望。

但是 ACL *passwords* 并不是真正的密码。它们是服务器和客户端之间共享的秘密，因为密码不是人类使用的身份验证令牌。例如：

*   没有长度限制, 密码只会在某些客户端软件中记住。在这种情况下, 没有人需要召回密码。
*   ACL 密码不保护任何其他内容。例如, 它永远不会是某些电子邮件帐户的密码。
*   通常, 当您能够访问散列密码本身时, 通过完全访问给定服务器的Redis命令或损坏系统本身, 您已经可以访问密码所保护的内容：Redis实例稳定性及其包含的数据。

出于这个原因，放慢密码认证，以使用一种利用时间和空间的算法来使密码破解变得困难，是一个非常糟糕的选择。相反，我们建议生成强密码，这样即使有哈希值，也没有人能够使用字典或暴力攻击来破解它。为此，有一个特殊的 ACL 命令“ACL GENPASS”，它使用系统加密伪随机生成器生成密码：

    > ACL GENPASS
    "dd721260bfe1b3d9601e7fbab36de6d04e2e67b0ef1c53de59d45950db0dd3cc"

该命令输出转换为 64 字节字母数字字符串的 32 字节（256 位）伪随机字符串。这足够长以避免攻击，也足够短以易于管理、剪切和粘贴、存储等。这是您应该用来生成 Redis 密码的内容。

## 使用外部 ACL 文件

有两种方法可以将用户存储在 Redis 配置中：

1.  用户可直接在`redis.conf`文件。
2.  可以指定外部 ACL 文件。

这两种方法*互不兼容*，因此 Redis 会要求您使用其中一种。在 `redis.conf` 中指定用户适用于简单的用例。当需要定义多个用户时，在复杂的环境中，我们建议您改用 ACL 文件。

`redis.conf` 内部和外部 ACL 文件中使用的格式完全相同，因此从一种切换到另一种很简单，如下所示：

    user <username> ... acl rules ...

例如：

    user worker +@list +@connection ~jobs:* on >ffa9203c493aa99

当您想使用外部 ACL 文件时，您需要指定名为 `aclfile` 的配置指令，如下所示：

    aclfile /etc/redis/users.acl

当您直接在 `redis.conf` 文件中指定几个用户时，可以使用 `CONFIG REWRITE` 通过重写将新用户配置存储在文件中。

但是, 外部 ACL 文件功能更强大。您可以执行以下操作：

*   用`ACL LOAD`如果您手动修改了 ACL 文件, 并且希望 Redis 重新加载新配置。请注意, 此命令能够加载文件*仅当正确指定了所有用户时*.否则, 将向用户报告错误, 并且旧配置将保持有效。
*   用`ACL SAVE`将当前 ACL 配置保存到 ACL 文件。

请注意，`CONFIG REWRITE` 不会同时触发 `ACL SAVE`。使用 ACL 文件时，配置和 ACL 分开处理。

## 哨兵和副本的 ACL 规则

如果您不想为 Redis 副本和 Redis Sentinel 实例提供对您的 Redis 实例的完全访问权限，以下是必须允许的一组命令，以便一切正常工作。

对于 Sentinel, 允许用户在主实例和副本实例中访问以下命令：

* AUTH, CLIENT, SUBSCRIBE, SCRIPT, PUBLISH, PING, INFO, MULTI, SLAVEOF, CONFIG, CLIENT, EXEC。

Sentinel 不需要访问数据库中的任何密钥, 但确实使用 Pub/Sub, 因此 ACL 规则如下 (注意：`AUTH`不需要, 因为它总是被允许的) ：

    ACL SETUSER sentinel-user on >somepassword allchannels +multi +slaveof +ping +exec +subscribe +config|rewrite +role +publish +info +client|setname +client|kill +script|kill

Redis 副本需要允许在主实例上执行以下命令：

*   PSYNC,  REPLCONF,  PING

无需访问任何密钥, 因此这转换为以下规则：

    ACL setuser replica-user on >somepassword +psync +replconf +ping

请注意, 您无需配置副本即可允许主服务器能够执行任何一组命令。从副本的角度来看, 主服务器始终被认证为 root 用户。
