集`field`存储在`key`自`value`, 仅当`field`不
但存在。
如果`key`不存在, 则会创建一个保存哈希值的新密钥。
如果`field`已存在, 此操作不起作用。

@return

@integer回复, 具体而言：

*   `1`如果`field`是哈希中的新字段, 并且`value`已设置。
*   `0`如果`field`哈希中已存在, 但未执行任何操作。

@examples

```cli
HSETNX myhash field "Hello"
HSETNX myhash field "World"
HGET myhash field
```
