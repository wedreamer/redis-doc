设置或清除位位于*抵消*中存储于 的字符串值*钥匙*.

位是设置的还是清除的，具体取决于*价值*，可以是 0 或
1\.

什么时候*钥匙*不存在，则创建新的字符串值。
绳子被放大以确保它可以保持在*抵消*.
这*抵消*参数必须大于或等于 0，并且小于
大于 2^32（这会将位图限制为 512MB）。
当字符串位于*钥匙*增长，添加的位设置为 0。

**警告**：设置最后一个可能的位 （*抵消*等于 2^32 -1） 和
存储在*钥匙*尚未保存字符串值，或保存
字符串值小，Redis需要分配所有中间内存，可以
阻止服务器一段时间。
在 2010 年的 MacBook Pro 上，设置位号 2^32 -1（512MB 分配）需要
\~300ms，设置位号 2^30 -1（128MB 分配）需要 ~80ms，设置位
数字 2^28 -1（32MB 分配）需要大约 30 毫秒，设置位号 2^26 -1（8MB）
分配）需要大约 8 毫秒。
请注意，完成第一次分配后，后续调用`SETBIT`为
一样*钥匙*将没有分配开销。

@return

@integer回复：存储在*抵消*.

@examples

```cli
SETBIT mykey 7 1
SETBIT mykey 7 0
GET mykey
```

## 模式：访问整个位图

在某些情况下，您需要一次设置单个位图的所有位，以便
将其初始化为默认非零值时的示例。这是可能的
这可以通过多次调用`SETBIT`命令，每个需要
被设置。但是，作为优化，您可以使用单个`SET`要设置的命令
整个位图。

位图不是实际的数据类型，而是一组面向位的操作
在字符串类型上定义（有关详细信息，请参阅
[“数据类型简介”页的位图部分][ti]).这意味着
位图可以与字符串命令一起使用，最重要的是与`SET`和
`GET`.

由于 Redis 的字符串是二进制安全的，因此位图被简单地编码为字节。
流。字符串的第一个字节对应于偏移量 0..7
位图、8..15 范围的第二个字节，依此类推。

例如，设置几个位后，获取位图的字符串值
将如下所示：

    > SETBIT bitmapsarestrings 2 1
    > SETBIT bitmapsarestrings 3 1
    > SETBIT bitmapsarestrings 5 1
    > SETBIT bitmapsarestrings 10 1
    > SETBIT bitmapsarestrings 11 1
    > SETBIT bitmapsarestrings 14 1
    > GET bitmapsarestrings
    "42"

通过获取位图的字符串表示形式，客户端可以分析
通过使用本机位操作提取位值来响应的字节
本机编程语言。对称地，也可以设置一个完整的
通过在客户端中执行位到字节编码并调用`SET`
替换为生成的字符串。

[ti]: /topics/data-types-intro#bitmaps

## 模式：设置多个位

`SETBIT`擅长设置单个位，并且可以在以下情况下多次调用
需要设置多个位。要优化此操作，您可以替换
倍数`SETBIT`通过对可变参数的单个调用进行调用`BITFIELD`命令
以及类型字段的使用`u1`.

例如，上面的示例可以替换为：

    > BITFIELD bitsinabitmap SET u1 2 1 SET u1 3 1 SET u1 5 1 SET u1 10 1 SET u1 11 1 SET u1 14 1

## 高级模式：访问位图范围

也可以使用`GETRANGE`和`SETRANGE`字符串命令
有效地访问位图中的一系列位偏移量。下面是一个示例
在惯用的 Redis Lua 脚本中实现，可以使用`EVAL`
命令：

    --[[
    Sets a bitmap range

    Bitmaps are stored as Strings in Redis. A range spans one or more bytes,
    so we can call `SETRANGE` when entire bytes need to be set instead of flipping
    individual bits. Also, to avoid multiple internal memory allocations in
    Redis, we traverse in reverse.
    Expected input:
      KEYS[1] - bitfield key
      ARGV[1] - start offset (0-based, inclusive)
      ARGV[2] - end offset (same, should be bigger than start, no error checking)
      ARGV[3] - value (should be 0 or 1, no error checking)
    ]]--

    -- A helper function to stringify a binary string to semi-binary format
    local function tobits(str)
      local r = ''
      for i = 1, string.len(str) do
        local c = string.byte(str, i)
        local b = ' '
        for j = 0, 7 do
          b = tostring(bit.band(c, 1)) .. b
          c = bit.rshift(c, 1)
        end
        r = r .. b
      end
      return r
    end

    -- Main
    local k = KEYS[1]
    local s, e, v = tonumber(ARGV[1]), tonumber(ARGV[2]), tonumber(ARGV[3])

    -- First treat the dangling bits in the last byte
    local ms, me = s % 8, (e + 1) % 8
    if me > 0 then
      local t = math.max(e - me + 1, s)
      for i = e, t, -1 do
        redis.call('SETBIT', k, i, v)
      end
      e = t
    end

    -- Then the danglings in the first byte
    if ms > 0 then
      local t = math.min(s - ms + 7, e)
      for i = s, t, 1 do
        redis.call('SETBIT', k, i, v)
      end
      s = t + 1
    end

    -- Set a range accordingly, if at all
    local rs, re = s / 8, (e + 1) / 8
    local rl = re - rs
    if rl > 0 then
      local b = '\255'
      if 0 == v then
        b = '\0'
      end
      redis.call('SETRANGE', k, rs, string.rep(b, rl))
    end

**注意：**从位图获取一系列位偏移量的实现是
留给读者作为练习。
