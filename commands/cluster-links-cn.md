Redis 集群中的每个节点都与集群中的每个对等体维护一对长期存在的 TCP 链路：一个用于向对等方发送出站消息，另一个用于从对等方接收入站消息。

`CLUSTER LINKS`将所有此类对等链接的信息输出为数组，其中每个数组元素都是一个映射，其中包含单个链接的属性及其值。

@examples

下面是一个示例输出：

    > CLUSTER LINKS
    1)  1) "direction"
        2) "to"
        3) "node"
        4) "8149d745fa551e40764fecaf7cab9dbdf6b659ae"
        5) "create-time"
        6) (integer) 1639442739375
        7) "events"
        8) "rw"
        9) "send-buffer-allocated"
       10) (integer) 4512
       11) "send-buffer-used"
       12) (integer) 0
    2)  1) "direction"
        2) "from"
        3) "node"
        4) "8149d745fa551e40764fecaf7cab9dbdf6b659ae"
        5) "create-time"
        6) (integer) 1639442739411
        7) "events"
        8) "r"
        9) "send-buffer-allocated"
       10) (integer) 0
       11) "send-buffer-used"
       12) (integer) 0

每个映射都由相应群集链接的以下属性及其值组成：

1.  `direction`：此链接由本地节点建立`to`对等体，或被本地节点接受`from`对等体。
2.  `node`：对等体的节点 ID。
3.  `create-time`：链接的创建时间。（在`to`link，这是本地节点创建 TCP 链路的时间，而不是实际建立 TCP 链路的时间。
4.  `events`：当前为链接注册的事件。`r`表示可读事件，`w`表示可写事件。
5.  `send-buffer-allocated`：分配链路发送缓冲区的大小，用于缓冲指向对等方的传出消息。
6.  `send-buffer-used`：当前保存数据（消息）的链接发送缓冲区部分的大小。

@return

@array回复：一个映射数组，其中每个映射都包含群集链接的各种属性及其值。
