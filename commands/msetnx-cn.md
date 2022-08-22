将给定的键设置为其各自的值。
`MSETNX`根本不会执行任何操作, 即使只有一个键已经
存在。

由于这种语义`MSETNX`可用于设置不同的键
表示唯一逻辑对象的不同字段, 以确保
设置所有字段或根本不设置任何字段。

`MSETNX`是原子的, 因此所有给定的键都是一次设置的。
客户端无法看到某些密钥已更新, 而
其他保持不变。

@return

@integer回复, 具体而言：

*   `1`如果设置了所有键。
*   `0`如果未设置任何键 (至少存在一个键) 。

@examples

```cli
MSETNX key1 "Hello" key2 "there"
MSETNX key2 "new" key3 "world"
MGET key1 key2 key3
```
