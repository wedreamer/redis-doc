`CLUSTER SHARDS`返回有关集群分片的详细信息。
分片被定义为为同一组插槽提供服务并相互复制的节点的集合。
在给定时间, 分片可能只有一个主节点, 但可能有多个副本或没有副本。
分片可能不为任何插槽提供服务, 同时仍然具有副本。

此命令将替换`CLUSTER SLOTS`命令, 通过提供更有效和可扩展的群集表示形式。

该命令适合由 Redis 集群客户端库使用, 以便了解集群的拓扑结构。
客户端应在启动时发出此命令, 以便检索映射关联群集*哈希槽*包含实际节点信息。
此映射应用于将命令定向到可能为与给定命令关联的插槽提供服务的节点。
如果命令发送到错误的节点, 因为它收到了“-MOVED”重定向, 则此命令可用于更新群集的拓扑。

该命令返回一个分片数组, 每个分片包含两个字段, “槽”和“节点”。

“slots”字段是此分片提供的槽范围列表, 存储为一对整数, 表示范围的开始和结束槽。
例如, 如果节点拥有插槽 1、2、3、5、7、8 和 9, 则插槽范围将存储为 \[1-3]、\[5-5]、\[7-9]。
因此, 槽字段将由以下整数列表表示。

    1) 1) "slots"
       2) 1) (integer) 1
          2) (integer) 3
          3) (integer) 5
          4) (integer) 5
          5) (integer) 7
          6) (integer) 9

“节点”字段包含分片内所有节点的列表。
每个单独的节点都是描述该节点的属性映射。
某些属性是可选的, 将来可能会添加更多属性。
当前属性列表：

*   id：此特定节点的唯一节点 ID。
*   终结点：到达节点的首选终结点, 有关此字段的可能值的详细信息, 请参阅下文。
*   ip：要向其发送此节点的请求的 IP 地址。
*   主机名 (可选) ：要为此节点发送请求的已公布主机名。
*   端口 (可) ) ：节点的 TCP (非 T) S) 端口。将至少存在一个端口或 tls 端口。
*   tls 端口 (可) ) ：节点的 TLS 端口。将至少存在一个端口或 tls 端口。
*   role：此节点的复制角色。
*   复制偏移量：此节点的复制偏移量。此信息可用于将命令发送到最新的副本。
*   运行状况：任一`online`,`failed`或`loading`.此信息应用于确定应向哪些节点发送流量。这`loading`应使用运行状况状态来了解节点当前不符合为流量提供服务的条件, 但将来可能符合条件。

终结点与端口一起定义客户端应用于发送给定槽的请求的位置。
终结点的 NULL 值表示节点具有未知终结点, 客户端应连接到用于发送`CLUSTER SHARDS`命令, 但端口从命令返回。
当 Redis 节点位于 Redis 不知道其终端节点的负载均衡器后面时, 此未知终端节点配置非常有用。
设置哪个端点由`cluster-preferred-endpoint-type`配置。

@return

@array回复：哈希范围和分片节点映射的嵌套列表。

@examples

    > CLUSTER SHARDS
    1) 1) "slots"
       2) 1) (integer) 10923
          2) (integer) 11110
          3) (integer) 11113
          4) (integer) 16111
          5) (integer) 16113
          6) (integer) 16383
       3) "nodes"
       4) 1)  1) "id"
              2) "71f058078c142a73b94767a4e78e9033d195dfb4"
              3) "port"
              4) (integer) 6381
              5) "ip"
              6) "127.0.0.1"
              7) "role"
              8) "primary"
              9) "replication-offset"
             10) (integer) 1500
             11) "health"
             12) "online"
          2)  1) "id"
              2) "1461967c62eab0e821ed54f2c98e594fccfd8736"
              3) "port"
              4) (integer) 7381
              5) "ip"
              6) "127.0.0.1"
              7) "role"
              8) "replica"
              9) "replication-offset"
             10) (integer) 700
             11) "health"
             12) "fail"
    2) 1) "slots"
       2) 1) (integer) 5461
          2) (integer) 10922
       3) "nodes"
       4) 1)  1) "id"
              2) "9215e30cd4a71070088778080565de6ef75fd459"
              3) "port"
              4) (integer) 6380
              5) "ip"
              6) "127.0.0.1"
              7) "role"
              8) "primary"
              9) "replication-offset"
             10) (integer) 1200
             11) "health"
             12) "online"
          2)  1) "id"
              2) "877fa59da72cb902d0563d3d8def3437fc3a0196"
              3) "port"
              4) (integer) 7380
              5) "ip"
              6) "127.0.0.1"
              7) "role"
              8) "replica"
              9) "replication-offset"
             10) (integer) 1100
             11) "health"
             12) "loading"
    3) 1) "slots"
       2) 1) (integer) 0
          2) (integer) 5460
          3) (integer) 11111
          4) (integer) 11112
          3) (integer) 16112
          4) (integer) 16112
       3) "nodes"
       4) 1)  1) "id"
              2) "b7e9acc0def782aabe6b596f67f06c73c2ffff93"
              3) "port"
              4) (integer) 7379
              5) "ip"
              6) "127.0.0.1"
              7) "hostname"
              8) "example.com"
              9) "role"
             10) "replica"
             11) "replication-offset"
             12) "primary"
             13) "health"
             14) "online"
          2)  1) "id"
              2) "e2acf1a97c055fd09dcc2c0dcc62b19a6905dbc8"
              3) "port"
              4) (integer) 6379
              5) "ip"
              6) "127.0.0.1"
              7) "hostname"
              8) "example.com"
              9) "role"
             10) "replica"
             11) "replication-offset"
             12) (integer) 0
             13) "health"
             14) "loading"
