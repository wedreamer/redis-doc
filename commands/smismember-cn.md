返回是否每个`member`是存储在`key`.

对于每个`member`,`1`如果值是集合的成员，则返回，或者`0`如果元素不是集合的成员，或者如果`key`不存在。

@return

@array回复：表示给定元素的成员身份的列表，在同一
按要求订购。

@examples

```cli
SADD myset "one"
SADD myset "one"
SMISMEMBER myset "one" "notamember"
```
