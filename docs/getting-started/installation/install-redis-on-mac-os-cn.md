***

## 标题：“在 macOS 上安装 Redis”&#xA;链接标题：“在 macOS 上安装”&#xA;体重： 1&#xA;描述： 使用 Homebrew 在 macOS 上安装和启动 Redis

本指南介绍如何使用 Homebrew 在 macOS 上安装 Redis。Homebrew是在macOS上安装Redis的最简单方法。如果您希望从 macOS 上的源文件构建 Redis, 请参阅 \[从源安装 Redis]。

## 先决条件

首先, 请确保您已安装自制软件。从终端, 运行：

{{< highlight bash >}}
$ brew --version
{{< / 突出显示>}}

如果此命令失败, 您需要[按照自制软件安装说明进行操作](https://brew.sh/).

## 安装

从终端, 运行：

{{< highlight bash >}}
brew install redis
{{< / 突出显示>}}

这将在您的系统上安装 Redis。

## 在前台启动和停止 Redis

要测试 Redis 安装, 您可以运行`redis-server`从命令行执行：

{{< highlight bash >}}
redis-server
{{< / 突出显示>}}

如果成功, 您将看到 Redis 的启动日志, Redis 将在前台运行。

要停止 Redis, 请输入`Ctrl-C`.

### 使用已启动启动启动启动和停止 Redis

作为在前台运行 Redis 的替代方法, 您还可以使用`launchd`在后台启动该过程：

{{< highlight bash >}}
酿造服务开始重新
{{< / 突出显示>}}

这将启动 Redis 并在登录时重新启动它。您可以检查`launchd`通过运行以下命令来管理 Redis：

{{< highlight bash >}}
酿造服务信息
{{< / 突出显示>}}

如果服务正在运行, 您将看到如下所示的输出：

{{< highlight bash >}}
redis  (homebrew.mxcl.redis) 
运行： ✔
加载： ✔
用户： 米兰达
PID： 67975
{{< / 突出显示>}}

要停止该服务, 请运行：

{{< highlight bash >}}
酿造服务停止再度
{{< / 突出显示>}}

## 连接到 Redis

Redis 运行后, 您可以通过运行来测试它`redis-cli`:

{{< highlight bash >}}
redis-cli
{{< / 突出显示>}}

这将打开 Redis REPL。请尝试运行一些命令：

{{< highlight bash >}}
127.0.0.1：6379> lpush demos redis-macOS-demo
还行
127.0.0.1：6379> rpop 演示
“redis-macOS-demo”
{{< / 突出显示>}}

## 四. 今后的步骤

拥有正在运行的 Redis 实例后, 您可能希望：

*   试用 Redis CLI 教程
*   使用其中一个 Redis 客户端进行连接
