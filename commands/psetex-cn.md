`PSETEX`工作原理一模一样`SETEX`与过期的唯一区别
时间以毫秒而不是秒为单位指定。

@examples

```cli
PSETEX mykey 1000 "Hello"
PTTL mykey
GET mykey
```
