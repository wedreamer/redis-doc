***

## 标题： “在 Linux 上安装 Redis”&#xA;linkTitle： “Install on Linux”&#xA;体重： 1&#xA;描述： >&#xA;如何在 Ubuntu、RHEL 和 CentOS 上安装 Redis

大多数主要的Linux发行版都为Redis提供软件包。

## 在 Ubuntu/Debian 上安装

您可以从官方安装最近稳定版本的 Redis`packages.redis.io`APT 存储库。

{{% 警报标题=“先决条件” 颜色=“警告” %}}
如果你运行的是一个非常小的发行版（比如 Docker 容器），你可能需要安装`lsb-release`第一：

{{< highlight bash >}}
sudo apt install lsb-release”
{{< / 突出显示>}}
{{% /alert %}}

将存储库添加到<code>容易</code>索引，更新它，然后安装：

{{< highlight bash >}}
卷曲 -fsSL https://packages.redis.io/gpg |sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

echo “deb \[signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $（lsb_release -cs） main” |sudo tee /etc/apt/sources.list.d/redis.list

sudo apt-get update
sudo apt-get install redis
{{< / 突出显示>}}

## 从 Snapcraft 安装

这[Snapcraft商店](https://snapcraft.io/store)提供[Redis 软件包](https://snapcraft.io/redis)可以安装在支持 snap 的平台上。

要通过快照安装，请运行：

{{< highlight bash >}}
sudo snap install redis
{{< / 突出显示>}}

如果您的 Linux 当前未安装 snap，您可以按照中所述的说明进行安装[安装贴靠](https://snapcraft.io/docs/installing-snapd).
