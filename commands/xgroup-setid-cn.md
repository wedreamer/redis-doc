设置**上次交付的 ID**对于使用者组。

通常, 当使用组创建时, 将设置使用者组上次传递的 ID`XGROUP CREATE`.
这`XGROUP SETID`命令允许修改组的上次传递 ID, 而无需删除并重新创建组。
例如, 如果您希望使用者组中的使用者重新处理流中的所有消息, 则可能需要将其下一个 ID 设置为 0：

    XGROUP SETID mystream mygroup 0

可选`entries_read`参数可以指定, 以便为任意 ID 启用使用者组滞后跟踪。
任意 ID 是指不是流的第一个条目、其最后一个条目或零  (“0-0”)  ID 的 ID 的任何 ID。
这非常有用, 您可以确切地知道任意 ID (不包括) ) 和流的最后一个条目之间有多少个条目。
在这种情况下, `entries_read`可以设置为流的`entries_added`减去条目数。

@return

@simple字符串回复：`OK`关于成功。
