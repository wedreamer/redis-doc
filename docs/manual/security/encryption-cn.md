---
title: "TLS"
linkTitle: "TLS"
weight: 1
description: Redis TLS support
aliases:
    - /topics/encryption
---

从版本 6 开始, Redis 支持 SSL/TLS 作为可选功能
需要在编译时启用。

## 开始

### 建筑

要使用TLS支持进行构建, 您需要OpenSSL开发库 (例如
libssl-dev on Debian/Ubuntu) .

跑`make BUILD_TLS=yes`.

### 测试

要使用 TLS 运行 Redis 测试套件, 您需要对 TCL 的 TLS 支持 (即
`tcl-tls`Debian/Ubuntu 上的软件包) 。

1.  跑`./utils/gen-test-certs.sh`生成根 CA 和服务器
    证书。

2.  跑`./runtest --tls`或`./runtest-cluster --tls`运行 Redis 和 Redis
    TLS 模式下的群集测试。

### 手动运行

使用 TLS 模式手动运行 Redis 服务器 (假设`gen-test-certs.sh`是
调用, 以便示例证书/密钥可用) ：

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

为了支持 TLS, Redis 必须配置 X.509 证书和
私钥。此外, 还需要指定 CA 证书捆绑包
文件或路径, 以便在验证证书时用作受信任的根。自
支持基于 DH 的密码, 也可以配置 DH 参数文件。例如：

    tls-cert-file /path/to/redis.crt
    tls-key-file /path/to/redis.key
    tls-ca-cert-file /path/to/ca.crt
    tls-dh-params-file /path/to/redis.dh

### TLS 侦听端口

这`tls-port`配置指令允许在 上接受 SSL/TLS 连接
指定的端口。这是**另外**收听`port`对于 TCP
连接, 因此可以使用 TLS 和
同时进行非 TLS 连接。

您可以指定`port 0`以完全禁用非 TLS 端口。仅启用
在默认 Redis 端口上使用 TLS, 请使用：

    port 0
    tls-port 6379

### 客户端证书身份验证

默认情况下, Redis 使用相互 TLS, 并要求客户端使用
有效证书 (根据指定的受信任根 CA 进行身份验证) 
`ca-cert-file`或`ca-cert-dir`).

您可以使用`tls-auth-clients no`以禁用客户端身份验证。

### 复制

Redis 主服务器处理连接客户端和副本服务器
方式, 所以以上`tls-port`和`tls-auth-clients`指令适用于
复制链接也是如此。

在副本服务器端, 需要指定`tls-replication yes`自
将 TLS 用于与主服务器的传出连接。

### 簇

使用 Redis 集群时, 请使用`tls-cluster yes`为了启用 TLS
群集总线和跨节点连接。

### 哨兵

Sentinel 从通用 Redis 继承其网络配置
配置, 因此上述所有内容也适用于 Sentinel。

连接到主服务器时, 圣天诺将使用`tls-replication`
指令, 以确定是否需要 TLS 或非 TLS 连接。

此外, 完全相同`tls-replication`指令将确定 Sentinel 的
接受来自其他哨兵的连接的端口也将支持 TLS。那是
哨兵将配置`tls-port`当且仅当`tls-replication`已启用。

### 其他配置

其他 TLS 配置可用于控制 TLS 协议的选择
版本、密码和密码套件等请查阅自证
`redis.conf`了解更多信息。

### 性能注意事项

TLS 在通信堆栈中添加了一个层, 由于写入/读取 SSL 连接、加密/解密和完整性检查而产生开销。因此, 使用 TLS 会导致每个 Redis 实例可实现的吞吐量降低 (有关更多信息, 请参阅此[讨论](https://github.com/redis/redis/issues/7595)).

### 局限性

TLS 当前不支持 I/O 线程。
