---
title: "Getting started with Redis"
linkTitle: "Getting started"
weight: 1
description: >
    How to get up and running with Redis
aliases:
    - /docs/getting-started/tutorial
---

这是开始使用 Redis 的指南。您将学习如何安装、运行和试验 Redis 服务器进程。

## 安装 Redis

如何安装 Redis 取决于您的操作系统。请参阅以下最适合您需求的指南：

*   \[从源代码安装 Redis]（{{< ref “/docs/getting-started/installation/install-redis-from-source.md” >}}）
*   \[在 Linux 上安装 Redis]（{{< ref “/docs/getting-started/installation/install-redis-on-linux.md” >}}）
*   \[在 macOS 上安装 Redis]（{{< ref “/docs/getting-started/installation/install-redis-on-mac-os.md” >}}）
*   \[在 Windows 上安装 Redis]（{{< ref “/docs/getting-started/installation/install-redis-on-windows.md” >}}）

一旦您启动并运行了 Redis，就可以使用`redis-cli`，则可以继续执行以下步骤。

## 使用 CLI 探索 Redis

外部程序使用 TCP 套接字和 Redis 特定协议与 Redis 通信。此协议在 Redis 客户端库中针对不同的编程语言实现。但是，为了使 Redis 的黑客攻击更简单，Redis 提供了一个命令行实用程序，可用于向 Redis 发送命令。此程序称为 **redis-cli**.

为了检查 Redis 是否正常工作，首先要做的是发送一个 **ping** 使用 redis-cli 的命令：

    $ redis-cli ping
    PONG

运行 **redis-cli** 后跟命令名称及其参数，会将此命令发送到在端口 6379 的 localhost 上运行的 Redis 实例。您可以更改 使用的主机和端口`redis-cli`- 只需尝试`--help`选项以检查使用情况信息。

另一种有趣的跑步方式`redis-cli`没有参数：程序将以交互模式启动。您可以键入不同的命令并查看其回复。

    $ redis-cli
    redis 127.0.0.1:6379> ping
    PONG
    redis 127.0.0.1:6379> set mykey somevalue
    OK
    redis 127.0.0.1:6379> get mykey
    "somevalue"

此时，您可以与 Redis 交谈。现在是时候暂停一下本教程并开始[15 分钟介绍 Redis 数据类型](https://redis.io/topics/data-types-intro)为了学习一些 Redis 命令。否则，如果您已经知道一些基本的 Redis 命令，则可以继续阅读。

# 保护 Redis

默认情况下，Redis 绑定到**所有接口**并且没有身份验证。如果您在非常受控的环境中使用 Redis，则与外部互联网和一般来自攻击者，这很好。但是，如果未硬化的 Redis 是暴露在互联网上，这是一个很大的安全问题。如果您不能 100% 确定您的环境已得到适当的保护，请检查以下步骤，以使 Redis 更安全，这些步骤是按照提高安全性的顺序登记的。

1.  确保 Redis 用于侦听连接的端口（默认情况下为 6379，如果在集群模式下运行 Redis，则另外为 16379，加上 Sentinel 的端口 26379）已设置防火墙，因此无法从外部世界联系 Redis。
2.  使用配置文件，其中`bind`指令的设置是为了保证 Redis 仅侦听您正在使用的网络接口。例如，如果您仅从同一台计算机本地访问 Redis，则仅环回接口 （127.0.0.1），依此类推。
3.  使用`requirepass`选项，以便添加额外的安全层，以便客户端需要使用`AUTH`命令。
4.  用[嗅探](http://www.tarsnap.com/spiped.html)或其他 SSL 隧道软件，以便在您的环境需要加密时加密 Redis 服务器和 Redis 客户端之间的流量。

请注意，Redis 实例暴露在互联网上，没有任何安全性[非常容易利用](http://antirez.com/news/96)，因此请确保您了解上述内容并应用**至少**防火墙层。防火墙就位后，请尝试使用`redis-cli`从外部主机为了证明自己，实例实际上无法访问。

# 从应用程序中使用 Redis

当然，仅从命令行界面使用Redis是不够的，因为目标是从您的应用程序中使用它。为此，您需要下载并安装适用于您的编程语言的 Redis 客户端库。
你会发现一个[本页中不同语言的完整客户端列表](https://redis.io/clients).

例如，如果您碰巧使用Ruby编程语言，我们的最佳建议是使用[Redis-rb](https://github.com/redis/redis-rb)客户。
您可以使用以下命令安装它**gem install redis**.

这些指令是特定于 Ruby 的，但实际上许多库客户端用于流行的语言看起来非常相似：你创建一个 Redis 对象并执行命令调用方法。使用 Ruby 的简短交互式示例：

    >> require 'rubygems'
    => false
    >> require 'redis'
    => true
    >> r = Redis.new
    => #<Redis client v4.5.1 for redis://127.0.0.1:6379/0>
    >> r.ping
    => "PONG"
    >> r.set('foo','bar')
    => "OK"
    >> r.get('foo')
    => "bar"

# Redis 持久性

你可以学习[Redis 持久性在此页面上的工作原理](https://redis.io/topics/persistence)，但是，对于快速入门来说，重要的是要了解的是，默认情况下，如果您使用默认配置启动 Redis， Redis 将仅不时自发地保存数据集（例如，如果数据中至少有100个更改，则至少在五分钟后），因此如果您希望数据库保留并在重新启动后重新加载，请确保调用**救**命令，每次要强制数据集快照时。否则，请确保使用**关闭**命令：

    $ redis-cli shutdown

这样，Redis将确保在退出之前将数据保存在磁盘上。
阅读[持久性页面](https://redis.io/topics/persistence)强烈建议使用，以便更好地了解 Redis 持久性的工作原理。

# 更正确地安装 Redis

从命令行运行Redis是可以的，只是为了破解一点或用于开发。但是，在某些时候，您将有一些实际的应用程序在真实的服务器上运行。对于这种用法，您有两种不同的选择：

*   使用屏幕运行 Redis。
*   使用初始化脚本以正确的方式将Redis安装在Linux框中，以便在重新启动后一切都将再次正常启动。

强烈建议使用初始化脚本进行正确安装。
以下说明可用于使用 Redis 版本 2.4 或更高版本附带的 init 脚本在基于 Debian 或 Ubuntu 的发行版中执行正确的安装。

我们假设您已经复制**redis-server**和**redis-cli**/usr/local/bin 下的可执行文件。

*   创建一个目录来存储您的 Redis 配置文件和数据：

          sudo mkdir /etc/redis
          sudo mkdir /var/redis

*   复制您将在 Redis 发行版中找到的 init 脚本，该脚本位于**utils**目录`/etc/init.d`.我们建议使用运行此 Redis 实例的端口的名称来调用它。例如：

          sudo cp utils/redis_init_script /etc/init.d/redis_6379

*   编辑初始化脚本。

          sudo vi /etc/init.d/redis_6379

确保修改**再度运动**相应地，您正在使用的端口。
pid 文件路径和配置文件名都取决于端口号。

*   将您在 Redis 分发版的根目录中找到的模板配置文件复制到`/etc/redis/`使用端口号作为名称，例如：

          sudo cp redis.conf /etc/redis/6379.conf

*   在内部创建目录`/var/redis`将用作此 Redis 实例的数据和工作目录：

          sudo mkdir /var/redis/6379

*   编辑配置文件，确保执行以下更改：
    *   设置**守护进程**更改为 yes（默认情况下设置为 no）。
    *   设置**皮德文件**自`/var/run/redis_6379.pid`（如果需要，请修改端口）。
    *   更改**港口**因此。在我们的示例中，不需要它，因为默认端口已经是6379。
    *   设置您的首选**对数级别**.
    *   设置**日志文件**自`/var/log/redis_6379.log`
    *   设置**迪尔**自`/var/redis/6379`（非常重要的一步！

*   最后，使用以下命令将新的 Redis init 脚本添加到所有默认运行级别：

          sudo update-rc.d redis_6379 defaults

大功告成！现在，您可以尝试使用以下命令运行实例：

    sudo /etc/init.d/redis_6379 start

确保一切按预期工作：

*   尝试使用 redis-cli ping 您的实例。
*   执行测试保存`redis-cli save`并检查转储文件是否正确存储在`/var/redis/6379/`（你应该找到一个名为`dump.rdb`).
*   检查您的 Redis 实例是否正确记录在日志文件中。
*   如果这是一台新机器，您可以毫无问题地尝试它，请确保重新启动后一切正常。

注意：在上面的说明中，我们跳过了许多您想要更改的 Redis 配置参数，例如，为了使用 AOF 持久性而不是 RDB 持久性，或者为了设置复制，等等。
请务必阅读示例[`redis.conf`](https://github.com/redis/redis/blob/6.2/redis.conf)文件（有大量注释）和其他文档，您可以在此网站中找到以获取更多信息。
