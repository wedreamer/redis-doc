---
title: Redis administration
linkTitle: Administration
weight: 1
description: Advice for configuring and managing Redis in production
aliases: [
    /topics/admin,
    /topics/admin.md,
    /manual/admin,
    /manual/admin.md,
]
---

## Redis 设置提示

### Linux

*   使用 Linux 操作系统部署 Redis。Redis 也在 OS X 上进行了测试， 并且不时在 FreeBSD 和 OpenBSD 系统上进行测试。但是，Linux 是执行大多数压力测试的地方，也是运行大多数生产部署的地方。

*   将 Linux 内核 `overcommit_memory` 设置设为 1。加`vm.overcommit_memory = 1`自`/etc/sysctl.conf`.然后，重新启动或运行该命令`sysctl vm.overcommit_memory=1`以激活设置。

*   要确保 Linux 内核功能 Transparent Giant Pages 不会影响 Redis 内存使用率和延迟，请使用以下命令：

`echo never > /sys/kernel/mm/transparent_hugepage/enabled`

### 内存

*   已确保已启用交换，并且交换文件大小等于系统上的内存量。如果 Linux 没有设置交换，并且您的 Redis 实例意外消耗了太多内存，则 Redis 可能会在内存不足时崩溃，或者 Linux 内核 OOM 杀手可能会终止 Redis 进程。启用交换后，您可以检测延迟峰值并对其执行操作。

*   设置显式`maxmemory`实例中的选项限制，以确保它将报告错误，而不是在接近达到系统内存限制时失败。请注意，`maxmemory`应通过计算 Redis 的开销（数据除外）和碎片开销来设置。因此，如果您认为自己有 10 GB 的可用内存，请将其设置为 8 或 9。

*   如果您在写入密集型应用程序中使用 Redis，则在磁盘上保存 RDB 文件或重写 AOF 日志时，Redis 最多可以使用正常使用的内存的 2 倍。使用的附加内存与保存过程中写入修改的内存页数成正比，因此它通常与在此期间触摸的键（或聚合类型项）数成正比。确保相应地调整内存大小。

*   查看`LATENCY DOCTOR`和`MEMORY DOCTOR`命令以帮助进行故障排除。

### 成像

*   在守护程序下运行时，使用`daemonize no`.

### 复制

*   设置一个与 Redis 使用的内存量成比例的不平凡的复制积压工作。积压工作允许副本更轻松地与主（主）实例同步。

*   如果使用复制，即使禁用了持久性，Redis 也会执行 RDB 保存。（这不适用于无盘复制。）如果主服务器上没有磁盘使用情况，请启用无盘复制。

*   如果使用复制，请确保主服务器启用了持久性，或者它不会在崩溃时自动重新启动。副本将尝试维护主服务器的精确副本，因此，如果主服务器使用空数据集重新启动，则副本也将被擦除。

### 安全

*   默认情况下，Redis 不需要任何身份验证，并侦听所有网络接口。如果您将 Redis 暴露在互联网上或攻击者可以访问它的其他地方，这是一个很大的安全问题。例如，请参见[此攻击](http://antirez.com/news/96)看看它有多危险。请查看我们的[安全页面](/topics/security)和[快速入门](/topics/quickstart)有关如何保护 Redis 的信息。

## 在 EC2 上运行 Redis

*   使用基于 HVM 的实例，而不是基于 PV 的实例。
*   不要使用旧的实例系列。例如，将 m3.medium 与 HVM 结合使用，而不是将 m1.medium 与 PV 结合使用。
*   需要谨慎处理将 Redis 持久性与 EC2 EBS 卷配合使用，因为有时 EBS 卷具有高延迟特征。
*   如果在副本与主节点同步时遇到问题，则可能需要尝试新的无盘复制。

## 在不停机的情况下升级或重新启动 Redis 实例

Redis 被设计为服务器中长时间运行的进程。您可以使用[配置设置命令](/commands/config-set).您还可以从 AOF 切换到 RDB 快照持久性，反之亦然，而无需重新启动 Redis。检查输出`CONFIG GET *`命令以了解更多信息。

有时需要重新启动，例如，将 Redis 进程升级到较新版本，或者当您需要修改 当前不支持的配置参数时`CONFIG`命令。

请按照以下步骤操作以避免停机。

*   将新的 Redis 实例设置为当前 Redis 实例的副本。为此，您需要一个不同的服务器，或者一个具有足够 RAM 的服务器来保持两个 Redis 实例同时运行。

*   如果使用单个服务器，请确保副本在与主实例不同的端口上启动，否则副本无法启动。

*   等待复制初始同步完成。检查副本的日志文件。

*   用`INFO`，确保主密钥和副本具有相同数量的密钥。用`redis-cli`以检查副本是否按预期工作并正在回复您的命令。

*   允许写入复制副本使用`CONFIG SET slave-read-only no`.

*   将所有客户端配置为使用新实例（副本）。请注意，您可能希望使用`CLIENT PAUSE`命令，以确保在切换期间没有客户端可以写入旧主服务器。

*   确认主服务器不再接收任何查询后（您可以使用[监视命令](/commands/monitor)），使用`REPLICAOF NO ONE`命令，然后关闭主节点。

如果您正在使用[Redis Sentinel](/topics/sentinel)或[Redis Cluster](/topics/cluster-tutorial)，升级到较新版本的最简单方法是先于升级一个副本。然后，您可以执行手动故障转移以将其中一个已升级的副本提升为主副本，最后升级最后一个副本。

***

**注意**

Redis 集群 4.0 在集群总线协议级别与 Redis 集群 3.2 不兼容，因此在这种情况下需要批量重启。但是，Redis 5 集群总线向后兼容 Redis 4。

***
