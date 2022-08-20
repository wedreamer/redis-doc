返回位值*抵消*中存储于 的字符串值*钥匙*.

什么时候*抵消*超出字符串长度，则假定字符串为
具有 0 位的连续空间。
什么时候*钥匙*不存在它被假定为空字符串，因此*抵消*是
始终超出范围，并且该值还假定为连续空间
0 位。

@return

@integer回复：存储在*抵消*.

@examples

```cli
SETBIT mykey 7 1
GETBIT mykey 0
GETBIT mykey 7
GETBIT mykey 100
```
