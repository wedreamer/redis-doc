在以下情况下返回`field`是存储在`key`.

@return

@integer回复, 具体而言：

*   `1`如果哈希包含`field`.
*   `0`如果哈希不包含`field`或`key`不存在。

@examples

```cli
HSET myhash field1 "foo"
HEXISTS myhash field1
HEXISTS myhash field2
```
