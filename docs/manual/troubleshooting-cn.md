---
title: "Troubleshooting Redis"
linkTitle: "Troubleshooting"
weight: 1
description: Problems with Redis? Start here.
aliases:
    - /topics/problems
---

本页试图帮助您在 Redis 出现问题时该怎么做。Redis项目的一部分是帮助遇到问题的人, 因为我们不喜欢让人们独自处理他们的问题。

*   如果您有**延迟问题**对于Redis, 在某种程度上似乎闲置了一段时间, 请阅读我们的[Redis 延迟故障排除指南](/topics/latency).
*   Redis稳定版本通常非常可靠, 但是在极少数情况下, 您**遇到崩溃**如果您提供调试信息, 开发人员可以提供更多帮助。请阅读我们的[调试 Redis 指南](/topics/debugging).
*   我们长期以来一直有用户在Redis上遇到崩溃的历史, 实际上事实证明, Redis是带有**内存损坏**.请使用以下方法测试您的内存**redis-server --test-memory**以防 Redis 在您的系统中不稳定。Redis内置内存测试速度快且相当可靠, 但如果可以的话, 您应该重新启动服务器并使用[内存测试86](http://memtest86.com).

对于所有其他问题, 请向[Redis Google Group](http://groups.google.com/group/redis-db).我们将很乐意为您提供帮助。

您还可以在[Redis Discord server](https://discord.gg/redis).

### Redis 3.0.x、2.8.x 和 2.6.x 中已知的关键错误列表

要查找关键错误列表, 请参阅更改日志：

*   [Redis 3.0 更新日志](https://raw.githubusercontent.com/redis/redis/3.0/00-RELEASENOTES).
*   [Redis 2.8 更新日志](https://raw.githubusercontent.com/redis/redis/2.8/00-RELEASENOTES).
*   [Redis 2.6 更新日志](https://raw.githubusercontent.com/redis/redis/2.6/00-RELEASENOTES).

检查*升级紧急性*每个补丁版本中的水平, 以便更容易地发现
包含重要修复程序的版本。

### 影响 Redis 的已知 Linux 相关 bug 列表。

*   Ubuntu 10.04 和 10.10 包含[错误](https://bugs.launchpad.net/ubuntu/+source/linux/+bug/666211)这可能会导致性能问题。不建议使用这些发行版附带的默认内核。据报道, Bug 影响了 EC2 实例, 但一些用户也提到了服务器的影响。
*   某些版本的 Xen 虚拟机管理程序报告分叉 ()  性能较差。看[延迟页面](/topics/latency)了解更多信息。
