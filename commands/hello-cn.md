切换到其他协议，可以选择进行身份验证和设置
连接的名称，或提供上下文客户端报告。

Redis 版本 6 及更高版本支持两种协议：旧协议、RESP2 和
Redis 6，RESP3中引入的新版本。RESP3 具有一定的优势，因为
当连接处于此模式时，Redis能够以更具语义性的方式进行回复
回复：例如，`HGETALL`将返回一个*地图类型*，因此是客户端库
实现不再需要提前知道将阵列转换为
在将其返回给调用方之前的哈希值。如需全面覆盖 RESP3，请
[检查此存储库](https://github.com/antirez/resp3).

在 Redis 6 中，连接以 RESP2 模式启动，因此实现 RESP2 的客户端会执行
不需要更新或更改。没有短期计划放弃对
RESP2，尽管将来的版本可能默认为 RESP3。

`HELLO`始终使用当前服务器和连接属性的列表进行回复，
例如：版本、加载的模块、客户端 ID、复制角色等。
在 Redis 6.2 中调用时没有任何参数及其对 RESP2 的默认使用
协议，回复如下所示：

    > HELLO
     1) "server"
     2) "redis"
     3) "version"
     4) "255.255.255"
     5) "proto"
     6) (integer) 2
     7) "id"
     8) (integer) 5
     9) "mode"
    10) "standalone"
    11) "role"
    12) "master"
    13) "modules"
    14) (empty array)

想要使用 RESP3 模式握手的客户端需要调用`HELLO`
命令，并将值“3”指定为`protover`参数，如下所示：

    > HELLO 3
    1# "server" => "redis"
    2# "version" => "6.0.0"
    3# "proto" => (integer) 3
    4# "id" => (integer) 10
    5# "mode" => "standalone"
    6# "role" => "master"
    7# "modules" => (empty array)

因为`HELLO`回复有用的信息，并给出`protover`是
可选或可设置为“2”，客户端库作者可以考虑使用此
命令而不是规范`PING`设置连接时。

当使用可选`protover`参数，此命令将切换
协议到指定版本，并且还接受以下选项：

*   `AUTH <username> <password>`：除了切换到指定的协议版本外，还可以直接对连接进行身份验证。这使得呼叫`AUTH`以前`HELLO`设置新连接时不需要。请注意，`username`可以设置为“默认”以针对不使用 ACL 但更简单的服务器进行身份验证`requirepass`版本 6 之前的 Redis 机制。
*   `SETNAME <clientname>`：这相当于调用`CLIENT SETNAME`.

@return

@array回复：服务器属性的列表。选择 RESP3 时，应答是映射而不是数组。如果`protover`请求不存在。
