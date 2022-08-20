返回存储在`key`，由
补偿`start`和`end`（两者均包括在内）。
可以使用负偏移来提供从末端开始的偏移
的字符串。
因此，-1 表示最后一个字符，-2 表示倒数第二个字符，依此类推。

该函数通过将结果范围限制为
字符串的实际长度。

@return

@bulk字符串回复

@examples

```cli
SET mykey "This is a string"
GETRANGE mykey 0 3
GETRANGE mykey -3 -1
GETRANGE mykey 0 -1
GETRANGE mykey 10 100
```
