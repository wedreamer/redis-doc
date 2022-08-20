此命令返回慢速日志中的当前条目数。

每当命令超过`slowlog-log-slower-than`配置指令。
慢速日志中的最大条目数由`slowlog-max-len`配置指令。
一旦 slog 日志达到其最大大小，每当创建新条目时，都会删除最旧的条目。
慢速日志可以使用`SLOWLOG RESET`命令。

@reply

@integer回复

慢速日志中的条目数。
