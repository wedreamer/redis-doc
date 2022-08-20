此命令复制存储在`source`键`destination`
钥匙。

默认情况下，`destination`键是在
连接。这`DB`选项允许指定备用逻辑数据库
目标键的索引。

该命令在`destination`键已存在。这
`REPLACE`选项删除了`destination`键，然后再将值复制到它。

@return

@integer回复，具体而言：

*   `1`如果`source`已复制。
*   `0`如果`source`未复制。

@examples

    SET dolly "sheep"
    COPY dolly clone
    GET clone
