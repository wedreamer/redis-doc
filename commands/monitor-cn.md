`MONITOR`是一个调试命令，它流回由 处理的每个命令
Redis 服务器。
它可以帮助了解数据库发生的情况。
此命令可以通过以下方式使用`redis-cli`和通过`telnet`.

按顺序查看服务器处理的所有请求的功能非常有用
在将 Redis 用作数据库和
分布式缓存系统。

    $ redis-cli monitor
    1339518083.107412 [0 127.0.0.1:60866] "keys" "*"
    1339518087.877697 [0 127.0.0.1:60866] "dbsize"
    1339518090.420270 [0 127.0.0.1:60866] "set" "x" "6"
    1339518096.506257 [0 127.0.0.1:60866] "get" "x"
    1339518099.363765 [0 127.0.0.1:60866] "eval" "return redis.call('set','x','7')" "0"
    1339518100.363799 [0 lua] "set" "x" "7"
    1339518100.544926 [0 127.0.0.1:60866] "del" "x"

用`SIGINT`（Ctrl-C） 停止`MONITOR`流运行方式`redis-cli`.

    $ telnet localhost 6379
    Trying 127.0.0.1...
    Connected to localhost.
    Escape character is '^]'.
    MONITOR
    +OK
    +1339518083.107412 [0 127.0.0.1:60866] "keys" "*"
    +1339518087.877697 [0 127.0.0.1:60866] "dbsize"
    +1339518090.420270 [0 127.0.0.1:60866] "set" "x" "6"
    +1339518096.506257 [0 127.0.0.1:60866] "get" "x"
    +1339518099.363765 [0 127.0.0.1:60866] "del" "x"
    +1339518100.544926 [0 127.0.0.1:60866] "get" "x"
    QUIT
    +OK
    Connection closed by foreign host.

手动发出`QUIT`或`RESET`用于停止`MONITOR`流运行
通过`telnet`.

## 监视器未记录的命令

出于安全考虑，不会记录任何管理命令
由`MONITOR`的输出和敏感数据在命令中进行编辑`AUTH`.

此外，命令`QUIT`也不会被记录。

## 运行监视器的成本

因为`MONITOR`流回**都**命令，它的使用是有代价的。
以下（完全不科学的）基准数据说明了成本
的运行`MONITOR`可以。

基准测试结果**没有** `MONITOR`运行：

    $ src/redis-benchmark -c 10 -n 100000 -q
    PING_INLINE: 101936.80 requests per second
    PING_BULK: 102880.66 requests per second
    SET: 95419.85 requests per second
    GET: 104275.29 requests per second
    INCR: 93283.58 requests per second

基准测试结果**跟** `MONITOR`运行 （`redis-cli monitor > /dev/null`):

    $ src/redis-benchmark -c 10 -n 100000 -q
    PING_INLINE: 58479.53 requests per second
    PING_BULK: 59136.61 requests per second
    SET: 41823.50 requests per second
    GET: 45330.91 requests per second
    INCR: 41771.09 requests per second

在这种特殊情况下，运行单个`MONITOR`客户端可以减少
吞吐量超过 50%。
跑得更多`MONITOR`客户端将进一步降低吞吐量。

@return

**非标准返回值**，只是将收到的命令转储到无限
流。

## 行为更改历史记录

*   `>= 6.0.0`:`AUTH`从命令的输出中排除。
*   `>= 6.2.0`: "`RESET`可以调用以退出监视模式。
*   `>= 6.2.4`: "`AUTH`,`HELLO`,`EVAL`,`EVAL_RO`,`EVALSHA`和`EVALSHA_RO`包含在命令的输出中。
