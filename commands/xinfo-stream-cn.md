此命令返回有关存储在`<key>`.

此命令提供的信息详细信息包括：

*   **长度**：流中的条目数 (请参阅`XLEN`)
*   **基数树键**：底层基数数据结构中的键数
*   **基数树节点**：底层基数数据结构中的节点数
*   **组**：为流定义的使用者组的数量
*   **上次生成的 ID**：添加到流中的最近最少条目的 ID
*   **最大删除条目 ID**：从流中删除的最大条目 ID
*   **已添加条目**：在流生存期内添加到流中的所有条目的计数
*   **首次入境**：流中第一个条目的 ID 和字段值元组
*   **最后一次输入**：流中最后一个条目的 ID 和字段值元组

可选`FULL`修饰符提供更详细的答复。
提供时, `FULL`回复包括**条目**数组, 由按升序排列的流条目 (ID 和字段值元组) 组成。
此外**组**也是一个数组, 对于每个使用者组, 它由报告的信息组成`XINFO GROUPS`和`XINFO CONSUMERS`.

这`COUNT`选项可用于限制返回的流和 PEL 条目的数量 (第一个`<count>`返回条) ) 。
默认值`COUNT`为 10 且 a`COUNT`0 表示将返回所有条目 (如果流中有很多条, , 则执行时间可能会很) ) 。

@return

@array回复：信息位列表

@examples

默认回复：

    > XINFO STREAM mystream
     1) "length"
     2) (integer) 2
     3) "radix-tree-keys"
     4) (integer) 1
     5) "radix-tree-nodes"
     6) (integer) 2
     7) "last-generated-id"
     8) "1638125141232-0"
     9) "max-deleted-entry-id"
    10) "0-0"
    11) "entries-added"
    12) (integer) 2
    13) "groups"
    14) (integer) 1
    15) "first-entry"
    16) 1) "1638125133432-0"
        2) 1) "message"
           2) "apple"
    17) "last-entry"
    18) 1) "1638125141232-0"
        2) 1) "message"
           2) "banana"

完整回复：

    > XADD mystream * foo bar
    "1638125133432-0"
    > XADD mystream * foo bar2
    "1638125141232-0"
    > XGROUP CREATE mystream mygroup 0-0
    OK
    > XREADGROUP GROUP mygroup Alice COUNT 1 STREAMS mystream >
    1) 1) "mystream"
       2) 1) 1) "1638125133432-0"
             2) 1) "foo"
                2) "bar"
    > XINFO STREAM mystream FULL
     1) "length"
     2) (integer) 2
     3) "radix-tree-keys"
     4) (integer) 1
     5) "radix-tree-nodes"
     6) (integer) 2
     7) "last-generated-id"
     8) "1638125141232-0"
     9) "max-deleted-entry-id"
    10) "0-0"
    11) "entries-added"
    12) (integer) 2
    13) "entries"
    14) 1) 1) "1638125133432-0"
           2) 1) "foo"
              2) "bar"
        2) 1) "1638125141232-0"
           2) 1) "foo"
              2) "bar2"
    15) "groups"
    16) 1)  1) "name"
            2) "mygroup"
            3) "last-delivered-id"
            4) "1638125133432-0"
            5) "entries-read"
            6) (integer) 1
            7) "lag"
            8) (integer) 1
            9) "pel-count"
           10) (integer) 1
           11) "pending"
           12) 1) 1) "1638125133432-0"
                  2) "Alice"
                  3) (integer) 1638125153423
                  4) (integer) 1
           13) "consumers"
           14) 1) 1) "name"
                  2) "Alice"
                  3) "seen-time"
                  4) (integer) 1638125153423
                  5) "pel-count"
                  6) (integer) 1
                  7) "pending"
                  8) 1) 1) "1638125133432-0"
                        2) (integer) 1638125153423
                        3) (integer) 1
    >
