---
title: "Redis security"
linkTitle: "Security"
weight: 1
description: Security model and features in Redis
aliases:
    - /topics/security
---

本文档从 Redis 的角度介绍了安全主题。它涵盖了 Redis 提供的访问控制、代码安全问题、可以通过选择恶意输入从外部触发的攻击以及其他类似主题。

对于与安全相关的联系人，请在 GitHub 上打开一个问题，或者当您认为保持通信的安全性非常重要时，请使用本文档末尾的 GPG 密钥。

## 安全模型

Redis 旨在由受信任环境中的受信任客户端访问。
这意味着通常将 Redis 实例直接暴露给 Internet 或者通常暴露给不受信任的客户端可以直接访问 Redis TCP 端口或 UNIX 套接字的环境不是一个好主意。

例如，在使用 Redis 作为数据库、缓存或消息传递系统实现的 Web 应用程序的常见上下文中，应用程序前端（Web 端）内的客户端将查询 Redis 以生成页面或执行请求的操作或由 Web 应用程序用户触发。

在这种情况下，Web 应用程序调解 Redis 和不受信任的客户端（访问 Web 应用程序的用户浏览器）之间的访问。

通常，对 Redis 的不可信访问应始终由实现 ACL、验证用户输入并决定对 Redis 实例执行哪些操作的层进行调解。

## 网络安全

除了网络中受信任的客户端外，任何人都应该拒绝访问 Redis 端口，因此运行 Redis 的服务器应该只能由使用 Redis 实现应用程序的计算机直接访问。

在直接暴露在互联网上的单台计算机的常见情况下，例如虚拟化 Linux 实例（Linode、EC2、...），应为 Redis 端口设置防火墙以防止来自外部的访问。客户端仍然可以使用环回接口访问 Redis。

请注意，可以通过在 **redis.conf** 文件中添加如下行来将 Redis 绑定到单个接口：

    bind 127.0.0.1

由于 Redis 的性质，未能从外部保护 Redis 端口可能会产生很大的安全影响。例如，外部攻击者可以使用单个“FLUSHALL”命令来删除整个数据集。

## 保护模式

不幸的是，许多用户未能保护 Redis 实例不被外部网络访问。许多实例只是通过公共 IP 暴露在互联网上。从 3.2.0 版本开始，Redis 在使用默认配置（绑定所有接口）执行时会进入一种称为 **保护模式** 的特殊模式，并且无需任何密码即可访问它。在这种模式下，Redis 只回复来自环回接口的查询，并回复从其他地址连接的客户端，并返回错误，说明问题以及如何正确配置 Redis。

我们希望保护模式能够严重减少未受保护的 Redis 实例在没有适当管理的情况下执行所导致的安全问题。但是系统管理员仍然可以忽略Redis给出的错误并禁用保护模式或手动绑定所有接口。

## 认证

虽然 Redis 不尝试实现访问控制，但它提供了一个微小的可选身份验证层，可以通过编辑 **redis.conf** 文件打开。

启用授权层后，Redis 将拒绝未经身份验证的客户端的任何查询。客户端可以通过发送 **AUTH** 命令和密码来验证自己。

密码由系统管理员在 redis.conf 文件中以明文形式设置。它应该足够长以防止暴力攻击，原因有两个：

*   Redis在提供查询方面非常快。外部客户端每秒可以测试许多密码。
*   Redis 密码存储在**redis.conf**文件和客户端配置内部。由于系统管理员不需要记住它, 因此密码可能很长。

身份验证层的目标是可选地提供一层冗余。如果为保护 Redis 免受外部攻击而实施的防火墙或任何其他系统出现故障，外部客户端仍然无法在不知道身份验证密码的情况下访问 Redis 实例。

由于 AUTH 命令与其他所有 Redis 命令一样，都是以未加密的方式发送的，因此它不能防止对网络具有足够访问权限以执行窃听的攻击者。

## TLS 支持

Redis 在所有通信通道上都可选支持 TLS，包括客户端连接、复制链接和 Redis 集群总线协议。

## 禁止特定命令

可以在 Redis 中禁止命令或将它们重命名为无法猜测的名称，以便普通客户端仅限于指定的命令集。

例如，虚拟化服务器提供商可能会提供托管 Redis 实例服务。在这种情况下，普通用户可能无法调用 Redis **CONFIG** 命令来更改实例的配置，但提供和删除实例的系统应该能够这样做。

在这种情况下，可以重命名或完全隐藏命令表中的命令。此功能可用作可在 redis.conf 配置文件中使用的语句。例如：

    rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52

在上面的示例中, **配置**命令已重命名为无法猜测的名称。 也可以通过将它重命名为空字符串来完全禁止它 (或任何其他命令) , 如以下示例所示：

    rename-command CONFIG ""

## 由来自外部客户端的恶意输入触发的攻击

即使没有外部访问实例，攻击者也可以从外部触发一类攻击。例如，攻击者可能会将数据插入 Redis，从而触发 Redis 内部实现的数据结构的病态（最坏情况）算法复杂性。

攻击者可以通过 Web 表单提供一组已知的字符串，这些字符串可以散列到散列表中的同一个桶中，以便将 O(1) 预期时间（平均时间）变为 O(N) 最差案子。这可能会消耗比预期更多的 CPU 并最终导致拒绝服务。

为了防止这种特定的攻击，Redis 对哈希函数使用了每次执行的伪随机种子。

Redis 使用 qsort 算法实现 SORT 命令。目前，该算法不是随机的，因此可以通过仔细选择正确的输入集来触发二次最坏情况行为。

## 字符串转义和 NoSQL 注入

Redis协议没有字符串转义的概念，所以在正常情况下使用普通的客户端库是不可能注入的.
该协议使用前缀长度的字符串并且是完全二进制安全的。

由于`EVAL`和`EVALSHA`命令执行的Lua脚本遵循相同的规则，这些命令也是安全的。

虽然这是一个奇怪的用例, 但应用程序应该避免从不受信任的源获取的字符串中编写Lua脚本的主体。

## 代码安全性

在经典的 Redis 设置中，允许客户端完全访问命令集，但访问实例绝不应该导致能够控制运行 Redis 的系统。

在内部，Redis 使用所有众所周知的做法来编写安全代码，以防止缓冲区溢出、格式错误和其他内存损坏问题。
但是，使用 **CONFIG** 命令控制服务器配置的能力允许客户端更改程序的工作目录和转储文件的名称。这允许客户端将 RDB Redis 文件写入随机路径。这是 [一个安全问题](http://antirez.com/news/96)，它可能会导致破坏系统和/或以与运行 Redis 的用户相同的用户身份运行不受信任的代码。

Redis 不需要 root 权限即可运行。建议以仅用于此目的的非特权 *redis* 用户身份运行它。

## GPG 密钥

    -----BEGIN PGP PUBLIC KEY BLOCK-----

    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
    =J9BD
    -----END PGP PUBLIC KEY BLOCK-----
