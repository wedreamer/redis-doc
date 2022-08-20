此命令创建一个由`<groupname>`对于存储在`<key>`.

每个组在给定流中都有一个唯一的名称。当已存在同名的使用者组时，该命令将返回`-BUSYGROUP`错误。

该命令的`<id>`参数从新组的角度指定流中最后传递的条目。
特殊 ID`$`表示流中最后一个条目的 ID，但您可以改为提供任何有效的 ID。
例如，如果希望组的使用者从头开始获取整个流，请使用零作为使用者组的起始 ID：

    XGROUP CREATE mystream mygroup 0

默认情况下，`XGROUP CREATE`命令坚持目标流存在，并在不存在时返回错误。
但是，您可以使用可选`MKSTREAM`子命令作为 后面的最后一个参数`<id>`以自动创建流（长度为 0）（如果该流不存在）：

    XGROUP CREATE mystream mygroup $ MKSTREAM

可选`entries_read`可以指定命名参数以启用任意 ID 的使用者组滞后跟踪。
任意 ID 是指不是流的第一个条目、其最后一个条目或零 （“0-0”） ID 的 ID 的任何 ID。
这非常有用，您可以确切地知道任意 ID（不包括它）和流的最后一个条目之间有多少个条目。
在这种情况下，`entries_read`可以设置为流的`entries_added`减去条目数。

@return

@simple字符串回复：`OK`关于成功。
