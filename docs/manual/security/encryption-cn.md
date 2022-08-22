---
title: "TLS"
linkTitle: "TLS"
weight: 1
description: Redis TLS support
aliases:
    - /topics/encryption
---

从版本 6 开始，Redis 支持 SSL/TLS 作为可选功能，需要在编译时启用。

## 开始

### 建筑

要使用 TLS 支持构建，您需要 OpenSSL 开发库（例如 Debian/Ubuntu 上的 libssl-dev）.

跑`make BUILD_TLS=yes`.

### 测试

要使用 TLS 运行 Redis 测试套件，您需要对 TCL 的 TLS 支持（即 Debian/Ubuntu 上的 `tcl-tls` 包）。

1.  跑`./utils/gen-test-certs.sh`生成根 CA 和服务器
    证书。

2.  跑`./runtest --tls`或`./runtest-cluster --tls`运行 Redis 和 Redis
    TLS 模式下的群集测试。

### 手动运行

使用 TLS 模式手动运行 Redis 服务器（假设调用了 `gen-test-certs.sh`，因此示例证书/密钥可用）：

    ./src/redis-server --tls-port 6379 --port 0 \
        --tls-cert-file ./tests/tls/redis.crt \
        --tls-key-file ./tests/tls/redis.key \
        --tls-ca-cert-file ./tests/tls/ca.crt

要连接到此 Redis 服务器, 请使用`redis-cli`:

    ./src/redis-cli --tls \
        --cert ./tests/tls/redis.crt \
        --key ./tests/tls/redis.key \
        --cacert ./tests/tls/ca.crt

### 证书配置

为了支持 TLS，Redis 必须配置 X.509 证书和私钥。此外，在验证证书时，需要指定一个 CA 证书捆绑文件或路径以用作受信任的根。要支持基于 DH 的密码，还可以配置 DH 参数文件。例如：

    tls-cert-file /path/to/redis.crt
    tls-key-file /path/to/redis.key
    tls-ca-cert-file /path/to/ca.crt
    tls-dh-params-file /path/to/redis.dh

### TLS 侦听端口

`tls-port` 配置指令允许在指定端口上接受 SSL/TLS 连接。这是除了监听 TCP 连接的“端口”之外的**，因此可以同时使用 TLS 和非 TLS 连接访问不同端口上的 Redis。

您可以指定 `port 0` 以完全禁用非 TLS 端口。要在默认 Redis 端口上仅启用 TLS，请使用：

    port 0
    tls-port 6379

### 客户端证书身份验证

默认情况下，Redis 使用双向 TLS 并要求客户端使用有效证书进行身份验证（根据 `ca-cert-file` 或 `ca-cert-dir` 指定的受信任根 CA 进行身份验证）.

您可以使用`tls-auth-clients no`以禁用客户端身份验证。

### 复制

Redis 主服务器以相同的方式处理连接客户端和副本服务器，因此上述 `tls-port` 和 `tls-auth-clients` 指令也适用于复制链接。

在副本服务器端，必须指定 `tls-replication yes` 以使用 TLS 进行到主服务器的传出连接。

### Cluster

使用 Redis 集群时，使用 `tls-cluster yes` 为集群总线和跨节点连接启用 TLS。

### 哨兵

Sentinel 从常见的 Redis 配置继承其网络配置，因此上述所有内容也适用于 Sentinel。

当连接到主​​服务器时，Sentinel 将使用 `tls-replication` 指令来确定是否需要 TLS 或非 TLS 连接。

此外，同样的 `tls-replication` 指令将确定接受来自其他 Sentinel 的连接的 Sentinel 端口是否也支持 TLS。也就是说，当且仅当启用 `tls-replication` 时，Sentinel 才会配置 `tls-port`。

### 其他配置

额外的 TLS 配置可用于控制 TLS 协议版本、密码和密码套件等的选择。请查阅自我记录的 `redis.conf` 了解更多信息。

### 性能注意事项

TLS 在通信堆栈中添加了一个层, 由于写入/读取 SSL 连接、加密/解密和完整性检查而产生开销。因此, 使用 TLS 会导致每个 Redis 实例可实现的吞吐量降低 (有关更多信息, 请参阅此[讨论](https://github.com/redis/redis/issues/7595)).

### 局限性

TLS 当前不支持 I/O 线程。
