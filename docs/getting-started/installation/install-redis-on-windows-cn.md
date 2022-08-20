***

## 标题： “在 Windows 上安装 Redis”&#xA;链接标题：“在 Windows 上安装”&#xA;体重： 1&#xA;描述： 在 Windows 上使用 Redis 进行开发

Redis在Windows上不受官方支持。但是，您可以按照以下说明在 Windows 上安装 Redis 以进行开发。

要在 Windows 上安装 Redis，您首先需要启用[WSL2](https://docs.microsoft.com/en-us/windows/wsl/install)（Windows Subsystem for Linux）。WSL2 允许您在 Windows 上本机运行 Linux 二进制文件。要使此方法正常工作，你需要运行 Windows 10 版本 2004 及更高版本或 Windows 11。

## 安装或启用 WSL2

微软提供[安装 WSL 的详细说明](https://docs.microsoft.com/en-us/windows/wsl/install).按照这些说明进行操作，并记下它安装的默认 Linux 发行版。本指南假定 Ubuntu。

## 安装 Redis

在 Windows 上运行 Ubuntu 后，您可以按照以下部分中详细介绍的步骤进行操作：[在 Ubuntu/Debian 上安装](install-redis-on-linux#install-on-ubuntu-debian)从官方安装最新的Redis稳定版本`packages.redis.io`APT 存储库。
将存储库添加到<code>容易</code>索引，更新它，然后安装：

{{< highlight bash >}}
卷曲 -fsSL https://packages.redis.io/gpg |sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

echo “deb \[signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $（lsb_release -cs） main” |sudo tee /etc/apt/sources.list.d/redis.list

sudo apt-get update
sudo apt-get install redis
{{< / 突出显示>}}

最后，像这样启动 Redis 服务器：

{{< highlight bash >}}
sudo service redis-server start
{{< / 突出显示>}}

## 连接到 Redis

您可以通过连接 Redis CLI 来测试您的 Redis 服务器是否正在运行：

{{< highlight bash >}}
redis-cli
127.0.0.1：6379> ping
乒乓
{{< / 突出显示>}}
