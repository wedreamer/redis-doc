递增指定的`field`存储在`key`, 并表示
浮点数, 由指定的`increment`.如果增量值
为负数, 结果是具有哈希字段值**递减**而不是递增。
如果该字段不存在, 则将其设置为`0`在执行操作之前。
如果出现下列情况之一, 则返回错误：

*   该字段包含错误类型的值 (不是字符串) 。
*   当前字段内容或指定的增量不可解析为
    双精度浮点数。

此命令的确切行为与`INCRBYFLOAT`
命令, 请参考文档`INCRBYFLOAT`进一步
信息。

@return

@bulk字符串回复：的值`field`在增量之后。

@examples

```cli
HSET mykey field 10.50
HINCRBYFLOAT mykey field 0.1
HINCRBYFLOAT mykey field -5
HSET mykey field 5.0e3
HINCRBYFLOAT mykey field 2.0e2
```

## 实现细节

该命令始终在复制链接和仅追加中传播
文件作为`HSET`操作, 使底层浮点的差异
数学实现不会成为不一致的根源。
