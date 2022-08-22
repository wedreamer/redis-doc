在以下情况下返回`key`存在。

用户应该知道, 如果在参数中多次提到相同的现有密钥, 则会多次计数。因此, 如果`somekey`存在`EXISTS somekey somekey`会返回2。

@return

@integer回复, 特别是指定为参数的键中存在的键数。

@examples

```cli
SET key1 "Hello"
EXISTS key1
EXISTS nosuchkey
SET key2 "World"
EXISTS key1 key2 nosuchkey
```
