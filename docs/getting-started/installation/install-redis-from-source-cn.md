***

## 标题：“从源代码安装 Redis”&#xA;链接标题：“从源代码安装”&#xA;体重： 5&#xA;描述： >&#xA;从源代码编译并安装 Redis

您可以在各种平台和操作系统（包括Linux和macOS）上从源代码编译和安装Redis。Redis 除了 C 编译器之外没有其他依赖项，并且`libc`.

## 下载源文件

Redis 源文件可在 \[此站点的下载页面] 上找到。您可以通过根据[redis-hashes git repository](https://github.com/redis/redis-hashes).

要从 Redis 下载站点获取 Redis 最新稳定版本的源文件，请运行：

{{< highlight bash >}}
wget https://download.redis.io/redis-stable.tar.gz
{{< / 突出显示>}}

## 编译 Redis

要编译 Redis，请先使用压缩包，更改为根目录，然后运行`make`:

{{< highlight bash >}}
tar -xzvf redis-stable.tar.gz
cd redis-stable
做
{{< / 突出显示>}}

如果编译成功，您将在`src`目录，包括：

*   **redis-server**：Redis 服务器本身
*   **redis-cli**是用于与 Redis 对话的命令行界面实用程序。

在 中安装这些二进制文件`/usr/local/bin`跑：

{{< highlight bash >}}
进行安装
{{< / 突出显示>}}

### 在前台启动和停止 Redis

安装后，您可以通过运行

{{< highlight bash >}}
redis-server
{{< / 突出显示>}}

如果成功，您将看到 Redis 的启动日志，Redis 将在前台运行。

要停止 Redis，请输入`Ctrl-C`.
