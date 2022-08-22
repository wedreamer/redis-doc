这`CLIENT GETNAME`返回由 设置的当前连接的名称`CLIENT SETNAME`.由于每个新连接在没有关联名称的情况下启动, 因此如果未分配任何名称, 则返回 null 批量回复。

@return

@bulk字符串回复：连接名称, 如果未设置名称, 则为空批量回复。
