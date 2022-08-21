---
title: "Redis command arguments"
linkTitle: "Command arguments"
weight: 1
description: How Redis commands expose their documentation programmatically
aliases:
    - /topics/command-arguments
---

这`COMMAND DOCS`命令返回有关可用 Redis 命令的以文档为中心的信息。
该命令返回的映射回复包括*参数*钥匙。
此键存储一个描述命令参数的数组。

中的每个元素*参数*array 是具有以下字段的映射：

*   **名字：**参数的名称，始终存在。
    给出参数的名称仅用于标识目的。
    在命令的语法呈现期间不显示它。
*   **类型：**参数的类型，始终存在。
    参数必须具有以下类型之一：
    *   **字符串：**字符串参数。
    *   **整数：**整数参数。
    *   **双：**双精度参数。
    *   **钥匙：**表示键名称的字符串。
    *   **模式：**表示类似 glob 的模式的字符串。
    *   **unix-time：**表示 Unix 时间戳的整数。
    *   **纯令牌：**参数是一个标记，表示保留关键字，可以提供，也可以不提供。
        不要与自由文本用户输入混淆。
    *   **其中之一**：参数是嵌套参数的容器。
        此类型允许在多个嵌套参数之间进行选择（请参阅`XADD`下面的示例）。
    *   **块：**参数是嵌套参数的容器。
        此类型允许对参数进行分组并应用属性（如*自选*） 添加到全部（请参阅`XADD`下面的示例）。
*   **key_spec_index：**此值可用于*钥匙*类型。
    它是命令中规范的从 0 开始的索引[主要规格][tr]对应于参数。
*   **令 牌**：参数（用户输入）本身之前的常量文本。
*   **总结：**参数的简短描述。
*   **因为：**参数的首次 Redis 版本（或对于模块命令，为模块版本）。
*   **deprecated_since：**弃用该命令的 Redis 版本（对于模块命令，则为模块版本）。
*   **标志：**参数标志的数组。
    可能的标志有：
    *   **自选**：表示参数是可选的（例如，*获取*的子句`SET`命令）。
    *   **倍数**：表示参数可以重复（例如*钥匙*参数`DEL`).
    *   **多令牌：**表示参数与其前面的标记可能重复（请参见`SORT`的`GET pattern`子句）。
*   **价值：**参数的值。
    对于参数类型，非*其中之一*和*块*，这是一个字符串，用于描述命令语法中的值。
    对于*其中之一*和*块*类型，这是一个嵌套参数数组，每个参数都是本节中所述的映射。

[tr]: /topics/key-specs

## 例

的修剪子句`XADD`，即`[MAXLEN|MINID [=|~] threshold [LIMIT count]]`，在顶层表示为*块*-类型参数。

它由四个嵌套参数组成：

1.  **修剪策略：**这个嵌套参数有一个*其中之一*具有两个嵌套参数的类型。
    每个嵌套参数，*麦克斯伦*和*迷你*，键入为*纯令牌*.
2.  **修剪运算符：**此嵌套参数是可选的*其中之一*具有两个嵌套参数的类型。
    每个嵌套参数，*=*和*~*，是一个*纯令牌*.
3.  **门槛：**这个嵌套参数是一个*字符串*.
4.  **计数：**此嵌套参数是可选的*整数*与*令 牌*(*限制*).

这是`XADD`的参数数组：

    1) 1) "name"
       2) "key"
       3) "type"
       4) "key"
       5) "value"
       6) "key"
    2)  1) "name"
        2) "nomkstream"
        3) "type"
        4) "pure-token"
        5) "token"
        6) "NOMKSTREAM"
        7) "since"
        8) "6.2"
        9) "flags"
       10) 1) optional
    3) 1) "name"
       2) "trim"
       3) "type"
       4) "block"
       5) "flags"
       6) 1) optional
       7) "value"
       8) 1) 1) "name"
             2) "strategy"
             3) "type"
             4) "oneof"
             5) "value"
             6) 1) 1) "name"
                   2) "maxlen"
                   3) "type"
                   4) "pure-token"
                   5) "token"
                   6) "MAXLEN"
                2) 1) "name"
                   2) "minid"
                   3) "type"
                   4) "pure-token"
                   5) "token"
                   6) "MINID"
                   7) "since"
                   8) "6.2"
          2) 1) "name"
             2) "operator"
             3) "type"
             4) "oneof"
             5) "flags"
             6) 1) optional
             7) "value"
             8) 1) 1) "name"
                   2) "equal"
                   3) "type"
                   4) "pure-token"
                   5) "token"
                   6) "="
                2) 1) "name"
                   2) "approximately"
                   3) "type"
                   4) "pure-token"
                   5) "token"
                   6) "~"
          3) 1) "name"
             2) "threshold"
             3) "type"
             4) "string"
             5) "value"
             6) "threshold"
          4)  1) "name"
              2) "count"
              3) "type"
              4) "integer"
              5) "token"
              6) "LIMIT"
              7) "since"
              8) "6.2"
              9) "flags"
             10) 1) optional
             11) "value"
             12) "count"
    4) 1) "name"
       2) "id_or_auto"
       3) "type"
       4) "oneof"
       5) "value"
       6) 1) 1) "name"
             2) "auto_id"
             3) "type"
             4) "pure-token"
             5) "token"
             6) "*"
          2) 1) "name"
             2) "id"
             3) "type"
             4) "string"
             5) "value"
             6) "id"
    5) 1) "name"
       2) "field_value"
       3) "type"
       4) "block"
       5) "flags"
       6) 1) multiple
       7) "value"
       8) 1) 1) "name"
             2) "field"
             3) "type"
             4) "string"
             5) "value"
             6) "field"
          2) 1) "name"
             2) "value"
             3) "type"
             4) "string"
             5) "value"
             6) "value"
