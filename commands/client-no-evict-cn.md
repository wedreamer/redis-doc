这`CLIENT NO-EVICT`命令将[客户驱逐](/topics/clients#client-eviction)当前连接的模式。

打开并配置客户端逐出后，即使我们高于配置的客户端逐出阈值，当前连接也将从客户端逐出过程中排除。

关闭后，当前客户端将被重新包含在要逐出的潜在客户池中（并在需要时被逐出）。

看[客户驱逐](/topics/clients#client-eviction)了解更多详情。

@return

@simple字符串回复：`OK`.
