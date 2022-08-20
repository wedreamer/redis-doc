***

## 标题： “Redis Security”&#xA;链接标题： “安全”&#xA;体重： 1&#xA;描述： Redis 中的安全模型和功能&#xA;别名：&#xA;\- /主题/安全

本文档从以下方面介绍了安全性主题：
雷迪斯的视图。它涵盖了Redis提供的访问控制，代码安全问题，
可以通过选择恶意输入从外部触发的攻击，以及
其他类似主题。

对于与安全相关的联系人，请在 GitHub 上或当您感觉到问题时打开问题
是真正重要的保持通信的安全性，使用
本文档末尾的 GPG 密钥。

## 安全模型

Redis 旨在由受信任环境中的受信任客户端访问。
这意味着公开 Redis 实例通常不是一个好主意。
直接到互联网，或者一般来说，到不受信任的环境
客户端可以直接访问 Redis TCP 端口或 UNIX 套接字。

例如，在使用 Redis 实现的 Web 应用程序的常见上下文中
作为数据库、缓存或消息传递系统，前端内部的客户端
（Web端）的应用将查询Redis生成页面或
以执行 Web 应用程序用户请求或触发的操作。

在这种情况下，Web 应用程序在 Redis 和
不受信任的客户端（访问 Web 应用程序的用户浏览器）。

通常，对 Redis 的不可信访问应该
始终由实现 ACL 的层进行调解，验证用户输入，
并决定对 Redis 实例执行哪些操作。

## 网络安全

应拒绝除受信任的客户端之外的所有人访问 Redis 端口
在网络中，因此运行 Redis 的服务器应该可以直接访问
仅由使用 Redis 实现应用程序的计算机执行。

在直接暴露在互联网上的一台计算机的常见情况下，例如
作为虚拟化的 Linux 实例（Linode、EC2 等），Redis 端口应该是
防火墙以防止从外部访问。客户仍然能够
使用环回接口访问 Redis。

请注意，可以通过添加一行将 Redis 绑定到单个接口
像下面到**redis.conf**文件：

    bind 127.0.0.1

如果不能从外部保护 Redis 端口，则可能会有很大的安全性
由于Redis的性质而产生的影响。例如，单个`FLUSHALL`命令可以被外部攻击者用来删除整个数据集。

## 保护模式

遗憾的是，许多用户无法防止 Redis 实例被访问
从外部网络。许多实例只是简单地暴露在
互联网与公共IP。从 3.2.0 版本开始，Redis 进入了一种特殊模式，称为**保护模式**当它是
使用默认配置（绑定所有接口）执行，并且
没有任何密码才能访问它。在此模式下，Redis 仅回复来自
环回接口，并回复从其他连接客户端的客户端
地址上有一个错误，用于解释问题以及如何配置
正确重新定位。

我们希望保护模式能够大大减少由此引起的安全问题
由未受保护的 Redis 实例执行，这些实例在未正确管理的情况下执行。然而
系统管理员仍然可以忽略 Redis 给出的错误，并且
禁用保护模式或手动绑定所有接口。

## 认证

虽然 Redis 不会尝试实现访问控制，但它提供
一小层可选身份验证，通过编辑
**redis.conf**文件。

启用授权层后，Redis 将拒绝以下列方式的任何查询
未经身份验证的客户端。客户端可以通过发送
**认证**命令后跟密码。

密码由系统管理员以明文形式设置
redis.conf file.它应该足够长以防止暴力攻击
原因有二：

*   Redis在提供查询方面非常快。外部客户端每秒可以测试许多密码。
*   Redis 密码存储在**redis.conf**文件和客户端配置内部。由于系统管理员不需要记住它，因此密码可能很长。

身份验证层的目标是选择性地提供
冗余。如果实施防火墙或任何其他系统来保护 Redis
由于外部攻击者失败，外部客户端仍将无法
在不了解身份验证密码的情况下访问 Redis 实例。

由于 AUTH 命令与所有其他 Redis 命令一样，是以未加密方式发送的，因此
无法抵御对网络具有足够访问权限的攻击者
进行窃听。

## TLS 支持

Redis 在所有通信通道上都具有对 TLS 的可选支持，包括
客户端连接、复制链接和 Redis 集群总线协议。

## 禁止特定命令

可以在 Redis 中禁止命令，也可以将它们重命名为不可猜测的命令
name，以便将普通客户端限制为一组指定的命令。

例如，虚拟化服务器提供商可能会提供托管 Redis 实例
服务。在这种情况下，普通用户可能无法
调用 Redis**配置**命令来更改实例的配置，
但是提供和删除实例的系统应该能够这样做。

在这种情况下，可以重命名或完全从
命令表。此功能可用作可以使用的语句
在 redis.conf 配置文件中。例如：

    rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52

在上面的示例中，**配置**命令已重命名为无法猜测的名称。 也可以通过将它重命名为空字符串来完全禁止它（或任何其他命令），如以下示例所示：

    rename-command CONFIG ""

## 由来自外部客户端的恶意输入触发的攻击

有一类攻击，攻击者甚至可以从外部触发
没有对实例的外部访问权限。例如，攻击者可能会将数据插入 Redis，从而触发病理性（最坏情况）
在 Redis 内部实现的数据结构的算法复杂性。

攻击者可以通过 Web 表单提供一组字符串，
已知在哈希表中散列到同一存储桶，以便将
O（1） 到 O（N） 最坏情况的预期时间（平均时间）。这可以消耗更多
CPU 超出预期，并最终导致拒绝服务。

为了防止这种特定的攻击，Redis使用每次执行的伪随机
哈希函数的种子。

Redis 使用 qsort 算法实现 SORT 命令。现在
该算法不是随机的，因此可以触发二次
通过仔细选择正确的输入集来执行最坏情况的行为。

## 字符串转义和 NoSQL 注入

Redis协议没有字符串转义的概念，因此注入
在正常情况下使用普通客户端库是不可能的。
该协议使用前缀长度的字符串，并且是完全二进制安全的。

由于 Lua 脚本由`EVAL`和`EVALSHA`命令跟随
同样的规则，这些命令也是安全的。

虽然这是一个奇怪的用例，但应用程序应该避免从不受信任的源获取的字符串中编写Lua脚本的主体。

## 代码安全性

在经典的 Redis 设置中，允许客户端完全访问命令集，
但是访问实例不应导致控制
运行 Redis 的系统。

在内部，Redis 使用所有众所周知的做法来编写安全代码
防止缓冲区溢出、格式错误和其他内存损坏问题。
但是，使用**配置**
命令允许客户端更改程序的工作目录和
转储文件的名称。这允许客户端写入 RDB Redis 文件
到随机路径。这是[安全问题](http://antirez.com/news/96)这可能会导致以与 Redis 相同的用户身份危害系统和/或运行不受信任的代码。

Redis 不需要 root 权限即可运行。建议
将其作为无特权运行*雷迪斯*仅用于此目的的用户。

## GPG 密钥

    -----BEGIN PGP PUBLIC KEY BLOCK-----

    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
    =J9BD
    -----END PGP PUBLIC KEY BLOCK-----
