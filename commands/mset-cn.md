将给定的键设置为其各自的值。
`MSET`用新值替换现有值，就像常规值一样`SET`.
看`MSETNX`如果不想覆盖现有值。

`MSET`是原子的，因此所有给定的键都是一次设置的。
客户端无法看到某些密钥已更新，而
其他保持不变。

@return

@simple字符串回复：始终`OK`因为`MSET`不能失败。

@examples

```cli
MSET key1 "Hello" key2 "World"
GET key1
GET key2
```
