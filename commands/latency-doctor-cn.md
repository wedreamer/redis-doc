这`LATENCY DOCTOR`命令报告与延迟相关的不同问题，并就可能的补救措施提出建议。

此命令是延迟监控中最强大的分析工具
框架，并且能够提供额外的统计数据，如平均值
延迟峰值、中位数偏差和人类可读之间的时间段
事件分析。对于某些事件，例如`fork`、附加信息
，就像系统分叉处理的速率一样。

这是您应该在 Redis 邮件列表中发布的输出，如果您
查找有关延迟相关问题的帮助。

@examples

    127.0.0.1:6379> latency doctor

    Dave, I have observed latency spikes in this Redis instance.
    You don't mind talking about it, do you Dave?

    1. command: 5 latency spikes (average 300ms, mean deviation 120ms,
        period 73.40 sec). Worst all time event 500ms.

    I have a few advices for you:

    - Your current Slow Log configuration only logs events that are
        slower than your configured latency monitor threshold. Please
        use 'CONFIG SET slowlog-log-slower-than 1000'.
    - Check your Slow Log to understand what are the commands you are
        running which are too slow to execute. Please check
        http://redis.io/commands/slowlog for more information.
    - Deleting, expiring or evicting (because of maxmemory policy)
        large objects is a blocking operation. If you have very large
        objects that are often deleted, expired, or evicted, try to
        fragment those objects into multiple smaller objects.

**注意：**医生的心理行为不稳定，因此我们建议您仔细与医生互动。

有关详细信息，请参阅[延迟监控框架页面][lm].

[lm]: /topics/latency-monitor

@return

@bulk字符串回复
