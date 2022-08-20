此命令返回属于`<groupname>`存储在 的流的使用者组`<key>`.

为组中的每个使用者提供以下信息：

*   **名字**：消费者的姓名
*   **待定**：客户端的挂起消息数，这些消息是已传递但尚未确认的消息
*   **怠**：自使用者上次与服务器交互以来经过的毫秒数

@reply

@array回复：消费者列表。

@examples

    > XINFO CONSUMERS mystream mygroup
    1) 1) name
       2) "Alice"
       3) pending
       4) (integer) 1
       5) idle
       6) (integer) 9104628
    2) 1) name
       2) "Bob"
       3) pending
       4) (integer) 1
       5) idle
       6) (integer) 83841983
