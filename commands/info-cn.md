这`INFO`命令返回有关服务器的信息和统计信息
易于计算机解析且易于人类阅读的格式。

可选参数可用于选择特定的信息部分：

*   `server`：有关 Redis 服务器的一般信息
*   `clients`：客户端连接部分
*   `memory`：内存消耗相关信息
*   `persistence`： RDB 和 AOF 相关信息
*   `stats`： 一般统计
*   `replication`：主/副本复制信息
*   `cpu`：CPU 消耗统计
*   `commandstats`：Redis 命令统计信息
*   `latencystats`：Redis 命令延迟百分位数分布统计信息
*   `cluster`：Redis 集群部分
*   `modules`：模块部分
*   `keyspace`： 数据库相关统计
*   `modules`：模块相关章节
*   `errorstats`：Redis 错误统计信息

它还可以采用以下值：

*   `all`：返回所有部分（不包括模块生成的部分）
*   `default`：仅返回默认的节集
*   `everything`： 包括`all`和`modules`

如果未提供参数，则`default`假定为选项。

@return

@bulk字符串回复：作为文本行的集合。

行可以包含节名称（以 # 字符开头）或属性。
所有属性都采用`field:value`终止者`\r\n`.

```cli
INFO
```

## 笔记

请注意，根据 Redis 的版本，某些字段已
已添加或删除。因此，一个健壮的客户端应用程序应该解析
通过跳过未知属性并正常处理此命令的结果
缺少字段。

以下是 Redis >= 2.4 的字段说明。

以下是**服务器**部分：

*   `redis_version`：Redis 服务器的版本
*   `redis_git_sha1`： Git SHA1
*   `redis_git_dirty`：Git 脏标志
*   `redis_build_id`：构建 ID
*   `redis_mode`：服务器的模式（“独立”、“哨兵”或“群集”）
*   `os`：托管 Redis 服务器的操作系统
*   `arch_bits`：体系结构（32 位或 64 位）
*   `multiplexing_api`：Redis 使用的事件循环机制
*   `atomicvar_api`：Redis使用的Atomicvar API
*   `gcc_version`：用于编译 Redis 服务器的 GCC 编译器版本
*   `process_id`：服务器进程的 PID
*   `process_supervised`：受监督系统（“新贵”、“系统化”、“未知”或“否”）
*   `run_id`：标识 Redis 服务器的随机值（供 Sentinel 使用）
    和群集）
*   `tcp_port`：TCP/IP 侦听端口
*   `server_time_usec`：基于纪元的系统时间，微秒级精度
*   `uptime_in_seconds`：自 Redis 服务器启动以来的秒数
*   `uptime_in_days`：以天为单位表示的值相同
*   `hz`：服务器的当前频率设置
*   `configured_hz`：服务器配置的频率设置
*   `lru_clock`：时钟每分钟递增，用于 LRU 管理
*   `executable`：服务器可执行文件的路径
*   `config_file`：配置文件的路径
*   `io_threads_active`：指示 I/O 线程是否处于活动状态的标志
*   `shutdown_in_milliseconds`：副本在完成关机序列之前赶上复制的最长时间。
    此字段仅在关机期间存在。

以下是**客户**部分：

*   `connected_clients`：客户端连接数（不包括连接数）
    从副本）
*   `cluster_connections`：使用的插槽数量的近似值
    集群的总线
*   `maxclients`：的值`maxclients`配置指令。这是
    总和的上限`connected_clients`,`connected_slaves`和
    `cluster_connections`.
*   `client_recent_max_input_buffer`：当前客户端连接中最大的输入缓冲区
*   `client_recent_max_output_buffer`：当前客户端连接中最大的输出缓冲区
*   `blocked_clients`：阻塞呼叫挂起的客户端数 （`BLPOP`,
    `BRPOP`,`BRPOPLPUSH`,`BLMOVE`,`BZPOPMIN`,`BZPOPMAX`)
*   `tracking_clients`：正在跟踪的客户端数 （`CLIENT TRACKING`)
*   `clients_in_timeout_table`：客户端超时表中的客户端数

以下是**记忆**部分：

*   `used_memory`：Redis 使用其分配的字节总数
    分配器（标准配置之一）**libc**,**杰马洛克**，或替代方法
    分配器，例如[**tcmalloc**][hcgcpgp])
*   `used_memory_human`：先前值的人类可读表示
*   `used_memory_rss`：Redis 分配的字节数，如
    操作系统（又名驻留集大小）。这是报告的数字
    工具，如`top(1)`和`ps(1)`
*   `used_memory_rss_human`：先前值的人类可读表示
*   `used_memory_peak`：Redis 消耗的峰值内存（以字节为单位）
*   `used_memory_peak_human`：先前值的人类可读表示
*   `used_memory_peak_perc`： 百分比`used_memory_peak`出
    `used_memory`
*   `used_memory_overhead`：服务器的所有开销的总和（以字节为单位）
    分配用于管理其内部数据结构
*   `used_memory_startup`：启动时 Redis 消耗的初始内存量
    以字节为单位
*   `used_memory_dataset`：数据集的大小（以字节为单位）
    (`used_memory_overhead`减去`used_memory`)
*   `used_memory_dataset_perc`： 百分比`used_memory_dataset`出
    网络内存使用情况 （`used_memory`减去`used_memory_startup`)
*   `total_system_memory`：Redis 主机具有的内存总量
*   `total_system_memory_human`：先前值的人类可读表示
*   `used_memory_lua`：Lua 引擎使用的字节数
*   `used_memory_lua_human`：先前值的人类可读表示
*   `used_memory_scripts`：缓存的 Lua 脚本使用的字节数
*   `used_memory_scripts_human`：先前值的人类可读表示
*   `maxmemory`：的值`maxmemory`配置指令
*   `maxmemory_human`：先前值的人类可读表示
*   `maxmemory_policy`：的值`maxmemory-policy`配置
    命令
*   `mem_fragmentation_ratio`： 比率`used_memory_rss`和`used_memory`.
    请注意，这不仅包括碎片，还包括其他进程开销（请参阅`allocator_*`指标），以及代码、共享库、堆栈等开销。
*   `mem_fragmentation_bytes`： 三角洲之间`used_memory_rss`和`used_memory`.
    请注意，当总碎片字节数较低（几兆字节）时，高比率（例如 1.5 及以上）并不表示存在问题。
*   `allocator_frag_ratio:`： 比率`allocator_active`和`allocator_allocated`.这是真正的（外部）碎片指标（不是`mem_fragmentation_ratio`).
*   `allocator_frag_bytes`之间的增量`allocator_active`和`allocator_allocated`.请参阅有关`mem_fragmentation_bytes`.
*   `allocator_rss_ratio`： 比率`allocator_resident`和`allocator_active`.这通常表示分配器可以并且可能很快就会释放回操作系统的页面。
*   `allocator_rss_bytes`： 三角洲之间`allocator_resident`和`allocator_active`
*   `rss_overhead_ratio`： 比率`used_memory_rss`（进程 RSS）和`allocator_resident`.这包括与分配器或堆无关的 RSS 开销。
*   `rss_overhead_bytes`： 三角洲之间`used_memory_rss`（进程 RSS）和`allocator_resident`
*   `allocator_allocated`：从分配器中分配的总字节数，包括内部碎片。通常与`used_memory`.
*   `allocator_active`：分配器活动页中的总字节数，这包括外部碎片。
*   `allocator_resident`：驻留在分配器中的总字节数 （RSS），这包括可以释放到操作系统的页面（通过`MEMORY PURGE`，或者只是等待）。
*   `mem_not_counted_for_evict`：未计入密钥逐出的已用内存。这基本上是瞬态副本和 AOF 缓冲区。
*   `mem_clients_slaves`：副本客户端使用的内存 - 从 Redis 7.0 开始，副本缓冲区与复制积压工作共享内存，因此当副本未触发内存使用量增加时，此字段可以显示 0。
*   `mem_clients_normal`：普通客户端使用的内存
*   `mem_cluster_links`：启用集群模式时，指向集群总线上对等方的链接使用的内存。
*   `mem_aof_buffer`：用于 AOF 和 AOF 重写缓冲区的瞬态存储器
*   `mem_replication_backlog`：复制积压工作使用的内存
*   `mem_total_replication_buffers`：复制缓冲区消耗的总内存 - 在 Redis 7.0 中添加。
*   `mem_allocator`：内存分配器，在编译时选择。
*   `active_defrag_running`： 当`activedefrag`，这表示碎片整理当前是否处于活动状态，以及它打算使用的 CPU 百分比。
*   `lazyfree_pending_objects`：等待释放的对象数（作为
    呼叫结果`UNLINK`或`FLUSHDB`和`FLUSHALL`与**异步**
    选项）
*   `lazyfreed_objects`：已延迟释放的对象数。

理想情况下，`used_memory_rss`值应仅略高于
`used_memory`.
当使用 rss >>时，很大的差异可能意味着存在（外部）内存碎片，这可以通过检查来评估
`allocator_frag_ratio`,`allocator_frag_bytes`.
当>> rss 使用时，这意味着 Redis 内存的一部分已被
操作系统：预计会出现一些明显的延迟。

因为 Redis 无法控制其分配映射到
内存页，高`used_memory_rss`通常是内存峰值的结果
用法。

当 Redis 释放内存时，内存将返回给分配器，并且
分配器可能会也可能不会将内存返回给系统。可能有
`used_memory`值和内存消耗作为
由操作系统报告。这可能是由于记忆已经
由Redis使用和发布，但未返回给系统。这
`used_memory_peak`值通常可用于检查这一点。

可以获得有关服务器内存的其他内省信息
通过引用`MEMORY STATS`命令和`MEMORY DOCTOR`.

以下是**坚持**部分：

*   `loading`：指示转储文件是否正在加载的标志
*   `async_loading`：当前在提供旧数据的同时异步加载复制数据集。这意味着`repl-diskless-load`已启用并设置为`swapdb`.在 Redis 7.0 中添加。
*   `current_cow_peak`：写入时复制内存的峰值大小（以字节为单位）
    当子分叉正在运行时
*   `current_cow_size`：写入时复制内存的大小（以字节为单位）
    当子分叉正在运行时
*   `current_cow_size_age`：年龄（以秒为单位）`current_cow_size`价值。
*   `current_fork_perc`：当前分叉进程的进度百分比。对于 AOF 和 RDB 分叉，其百分比为`current_save_keys_processed`出`current_save_keys_total`.
*   `current_save_keys_processed`：当前保存操作处理的键数
*   `current_save_keys_total`：当前保存操作开始时的键数
*   `rdb_changes_since_last_save`：自上次转储以来的更改数
*   `rdb_bgsave_in_progress`：指示 RDB 正在保存的标志
*   `rdb_last_save_time`：上次成功保存 RDB 的基于纪元的时间戳
*   `rdb_last_bgsave_status`：上次 RDB 保存操作的状态
*   `rdb_last_bgsave_time_sec`：中最后一个 RDB 保存操作的持续时间
    秒
*   `rdb_current_bgsave_time_sec`：正在进行的 RDB 保存操作的持续时间
    如果有的话
*   `rdb_last_cow_size`：写入时复制内存的大小（以字节为单位）
    上次 RDB 保存操作
*   `rdb_last_load_keys_expired`：上次加载 RDB 期间删除的易失性密钥数。在 Redis 7.0 中添加。
*   `rdb_last_load_keys_loaded`：上次加载 RDB 期间加载的密钥数。在 Redis 7.0 中添加。
*   `aof_enabled`：指示 AOF 日志记录已激活的标志
*   `aof_rewrite_in_progress`：指示 AOF 重写操作的标志是
    进行中
*   `aof_rewrite_scheduled`：指示 AOF 重写操作的标志
    将在正在进行的 RDB 保存完成后进行计划。
*   `aof_last_rewrite_time_sec`：中最后一次 AOF 重写操作的持续时间
    秒
*   `aof_current_rewrite_time_sec`：正在进行的 AOF 重写的持续时间
    操作（如果有）
*   `aof_last_bgrewrite_status`：上次 AOF 重写操作的状态
*   `aof_last_write_status`：上次写入 AOF 操作的状态
*   `aof_last_cow_size`：写入时复制内存的大小（以字节为单位）
    最后一个 AOF 重写操作
*   `module_fork_in_progress`：指示模块分叉正在进行中的标志
*   `module_fork_last_cow_size`：写入时复制内存的大小（以字节为单位）
    在最后一次模块分叉操作期间
*   `aof_rewrites`：自启动以来执行的 AOF 重写次数
*   `rdb_saves`：自启动以来执行的 RDB 快照数

`rdb_changes_since_last_save`指产生的操作数
自上次以来数据集中的某种变化`SAVE`或
`BGSAVE`被调用。

如果激活了 AOF，则将添加以下附加字段：

*   `aof_current_size`：AOF 当前文件大小
*   `aof_base_size`：最近启动或重写时的 AOF 文件大小
*   `aof_pending_rewrite`：指示 AOF 重写操作的标志
    将在正在进行的 RDB 保存完成后进行计划。
*   `aof_buffer_length`：AOF 缓冲区的大小
*   `aof_rewrite_buffer_length`：AOF 重写缓冲区的大小。请注意，此字段已在 Redis 7.0 中删除
*   `aof_pending_bio_fsync`：后台 I/O 中的 fsync 挂起作业数
    队列
*   `aof_delayed_fsync`：延迟的同步计数器

如果正在进行加载操作，将添加以下附加字段：

*   `loading_start_time`：基于纪元的加载开始时间戳
    操作
*   `loading_total_bytes`：总文件大小
*   `loading_rdb_used_mem`：已生成
    创建文件时的 RDB 文件
*   `loading_loaded_bytes`：已加载的字节数
*   `loading_loaded_perc`：以百分比表示的相同值
*   `loading_eta_seconds`：以秒为单位的ETA，即可完成加载

以下是**统计**部分：

*   `total_connections_received`：接受的连接总数
    服务器
*   `total_commands_processed`：服务器处理的命令总数
*   `instantaneous_ops_per_sec`：每秒处理的命令数
*   `total_net_input_bytes`：从网络读取的总字节数
*   `total_net_output_bytes`：写入网络的字节总数
*   `total_net_repl_input_bytes`：出于复制目的从网络读取的总字节数
*   `total_net_repl_output_bytes`：写入网络以进行复制的总字节数
*   `instantaneous_input_kbps`：网络的每秒读取速率（以 KB/秒为单位）
*   `instantaneous_output_kbps`：网络的每秒写入速率（以 KB/秒为单位）
*   `instantaneous_input_repl_kbps`：用于复制的网络每秒读取速率（以 KB/秒为单位）
*   `instantaneous_output_repl_kbps`：用于复制的网络每秒写入速率（以 KB/秒为单位）
*   `rejected_connections`：由于以下原因而被拒绝的连接数
    `maxclients`限制
*   `sync_full`：与复制副本完全重新同步的次数
*   `sync_partial_ok`：接受的部分重新同步请求数
*   `sync_partial_err`：被拒绝的部分重新同步请求数
*   `expired_keys`：密钥过期事件的总数
*   `expired_stale_perc`：密钥可能已过期的百分比
*   `expired_time_cap_reached_count`：活动到期周期提前停止的次数
*   `expire_cycle_cpu_milliseconds`：在活动到期周期上花费的累计时间
*   `evicted_keys`：由于`maxmemory`限制
*   `evicted_clients`：由于`maxmemory-clients`限制。在 Redis 7.0 中添加。
*   `total_eviction_exceeded_time`： 总时间`used_memory`大于`maxmemory`自服务器启动以来，以毫秒为单位
*   `current_eviction_exceeded_time`：此后的时间流逝`used_memory`最后一次升起`maxmemory`，以毫秒为单位
*   `keyspace_hits`：在主字典中成功查找键的次数
*   `keyspace_misses`：在主字典中查找键失败的次数
*   `pubsub_channels`：全球数量的发布/订阅频道与客户端
    订阅
*   `pubsub_patterns`：具有客户端的发布/订阅模式的全局数量
    订阅
*   `pubsubshard_channels`：具有客户端订阅的发布/订阅分片通道的全球数量。已在 Redis 7.0.3 中添加
*   `latest_fork_usec`：最新分叉操作的持续时间（以微秒为单位）
*   `total_forks`：自服务器启动以来的分叉操作总数
*   `migrate_cached_sockets`：打开的插槽数`MIGRATE`目的
*   `slave_expires_tracked_keys`：为到期目的而跟踪的密钥数
    （仅适用于可写副本）
*   `active_defrag_hits`：由活动执行的值重新分配数
    碎片整理过程
*   `active_defrag_misses`：由
    活动碎片整理过程
*   `active_defrag_key_hits`：已主动进行碎片整理的密钥数
*   `active_defrag_key_misses`：活动用户跳过的键数
    碎片整理过程
*   `total_active_defrag_time`：内存碎片的总时间超过限制，以毫秒为单位
*   `current_active_defrag_time`：自上次内存碎片超过限制以来经过的时间，以毫秒为单位
*   `tracking_total_keys`：服务器正在跟踪的密钥数
*   `tracking_total_items`：项目数，即 的客户端数之和
    正在跟踪的每个键
*   `tracking_total_prefixes`：服务器前缀表中跟踪的前缀数
    （仅适用于广播模式）
*   `unexpected_error_replies`：意外错误回复的数量，即类型
    来自 AOF 加载或复制的错误
*   `total_error_replies`：发出的错误回复总数，即
    拒绝的命令（命令执行前的错误）和
    失败的命令（命令执行中的错误）
*   `dump_payload_sanitizations`：转储有效负载深度完整性验证的总数（请参阅`sanitize-dump-payload`配置）。
*   `total_reads_processed`：处理的读取事件总数
*   `total_writes_processed`：处理的写入事件总数
*   `io_threaded_reads_processed`：主线程和 I/O 线程处理的读取事件数
*   `io_threaded_writes_processed`：主线程和 I/O 线程处理的写入事件数

以下是**复制**部分：

*   `role`：如果实例不是任何人的副本，则值为“主”;如果实例是某个主实例的副本，则值为“从属”。
    请注意，一个副本可以是另一个副本（链接复制）的主副本。
*   `master_failover_state`：正在进行的故障转移的状态（如果有）。
*   `master_replid`：Redis 服务器的复制 ID。
*   `master_replid2`：辅助复制 ID，用于故障转移后的 PSYNC。
*   `master_repl_offset`：服务器的当前复制偏移量
*   `second_repl_offset`：接受复制 ID 的偏移量
*   `repl_backlog_active`：指示复制积压工作处于活动状态的标志
*   `repl_backlog_size`：复制积压缓冲区的总大小（以字节为单位）
*   `repl_backlog_first_byte_offset`：复制的主偏移量
    积压缓冲区
*   `repl_backlog_histlen`：复制积压工作中数据的大小（以字节为单位）
    缓冲区

如果实例是副本，则提供以下附加字段：

*   `master_host`：主站的主机或 IP 地址
*   `master_port`：主侦听 TCP 端口
*   `master_link_status`：链接的状态（向上/向下）
*   `master_last_io_seconds_ago`：自上次交互以来的秒数
    与母版
*   `master_sync_in_progress`：指示主服务器正在同步到副本
*   `slave_read_repl_offset`：副本实例的读取复制偏移量。
*   `slave_repl_offset`：副本实例的复制偏移量
*   `slave_priority`：实例作为故障转移候选项的优先级
*   `slave_read_only`：指示复制副本是否为只读的标志
*   `replica_announced`：指示副本是否由 Sentinel 宣布的标志。

如果正在执行 SYNC 操作，则会提供以下附加字段：

*   `master_sync_total_bytes`：需要的字节总数
    转移。当大小未知时，此值可能为 0（例如，当
    这`repl-diskless-sync`使用配置指令）
*   `master_sync_read_bytes`：已传输的字节数
*   `master_sync_left_bytes`：同步完成之前剩余的字节数
    （当`master_sync_total_bytes`为 0）
*   `master_sync_perc`：百分比`master_sync_read_bytes`从
    `master_sync_total_bytes`，或使用
    `loading_rdb_used_mem`什么时候`master_sync_total_bytes`为 0
*   `master_sync_last_io_seconds_ago`：自上次传输 I/O 以来的秒数
    在同步操作期间

如果主服务器和副本之间的链路已关闭，则会提供一个附加字段：

*   `master_link_down_since_seconds`：自链接断开以来的秒数

始终提供以下字段：

*   `connected_slaves`：连接的副本数

如果服务器配置了`min-slaves-to-write`（或从 Redis 5 开始，使用`min-replicas-to-write`） 指令，则提供了一个附加字段：

*   `min_slaves_good_slaves`：当前认为良好的副本数

对于每个副本，将添加以下行：

*   `slaveXXX`：id、IP 地址、端口、状态、偏移、滞后

以下是**中央处理器**部分：

*   `used_cpu_sys`：Redis 服务器消耗的系统 CPU，即服务器进程的所有线程（主线程和后台线程）消耗的系统 CPU 的总和
*   `used_cpu_user`：Redis 服务器消耗的用户 CPU，即服务器进程的所有线程（主线程和后台线程）消耗的用户 CPU 的总和
*   `used_cpu_sys_children`：后台进程占用的系统 CPU
*   `used_cpu_user_children`：后台进程占用的用户 CPU
*   `used_cpu_sys_main_thread`：Redis 服务器主线程占用的系统 CPU
*   `used_cpu_user_main_thread`：Redis 服务器主线程消耗的用户 CPU

这**命令统计**部分提供基于命令类型的统计信息，
包括到达命令执行（未拒绝）的调用数，
这些命令消耗的总 CPU 时间，平均消耗的 CPU
每个命令执行，拒绝的调用数
（命令执行前的错误），以及失败调用的次数
（命令执行中的错误）。

对于每种命令类型，将添加以下行：

*   `cmdstat_XXX`:`calls=XXX,usec=XXX,usec_per_call=XXX,rejected_calls=XXX,failed_calls=XXX`

这**延迟统计**部分提供基于命令类型的延迟百分位数分布统计信息。

默认情况下，导出的延迟百分位数为 p50、p99 和 p999。
如果需要更改导出的百分位数，请使用`CONFIG SET latency-tracking-info-percentiles "50.0 99.0 99.9"`.

此部分需要启用延长延迟监视功能（默认情况下已启用）。
如果需要启用它，请使用`CONFIG SET latency-tracking yes`.

对于每种命令类型，将添加以下行：

*   `latency_percentiles_usec_XXX: p<percentile 1>=<percentile 1 value>,p<percentile 2>=<percentile 2 value>,...`

这**错误统计**部分可以跟踪 Redis 中发生的不同错误，
基于回复错误前缀 （ “-” 之后的第一个单词，直到第一个空格。例：`ERR`).

对于每种错误类型，将添加以下行：

*   `errorstat_XXX`:`count=XXX`

这**簇**部分当前仅包含一个唯一字段：

*   `cluster_enabled`：指示 Redis 集群已启用

这**模块**部分包含有关已加载模块的其他信息（如果模块提供）。本节中属性行的字段部分始终以模块名称为前缀。

这**键空间**部分提供了每个主词典的统计信息
数据库。
统计信息是密钥数和过期密钥数。

对于每个数据库，将添加以下行：

*   `dbXXX`:`keys=XXX,expires=XXX`

[hcgcpgp]: http://code.google.com/p/google-perftools/

**关于本手册页中使用的“奴隶”一词的说明**：从 Redis 5 开始，如果不是为了向后兼容，Redis 项目不再使用“从属”一词。不幸的是，在此命令中，单词 slave 是协议的一部分，因此只有当此 API 自然弃用时，我们才能删除此类事件。

**模块生成的部分**：从 Redis 6 开始，模块可以将其信息注入到`INFO`命令，默认情况下，即使`all`参数（它将包括已加载模块的列表，但不包括其生成的信息字段）。要获得这些，您必须使用`modules`参数或`everything`.,
