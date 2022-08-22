***
---
title: "Redis license"
linkTitle: "License"
weight: 5
description: >
    Redis license and trademark information
aliases:
    - /topics/license
---


Redis is **开源软件** 根据 **三条款 BSD 许可证**. Redis 的大部分源代码都是由 Salvatore Sanfilippo 和 Pieter Noordhuis 编写的, 并受版权保护。其他贡献者的列表可以在 git 历史记录中找到。

Redis商标和徽标归 Redis Ltd. 所有, 可以是按照[Redis Trademark Guidelines](/docs/about/trademark).

## 三条款 BSD 许可证

Redis 发行版中的每个文件 (下表中指定的第三方文件除外) 都包含以下许可证：

以源代码和二进制形式重新分发和使用, 有或没有只要满足以下条件, 允许修改：

*   源代码的再分发必须保留上述版权声明, 此条件列表和以下免责声明。

*   二进制形式的再分发必须复制上述版权通知, 此条件列表和以下免责声明随分发提供的文档和/或其他材料。

*   不得使用 Redis 的名称或其贡献者的名称认可或推广从本软件派生的产品, 而无需事先给予具体书面许可。

本软件由版权所有者和贡献者 "按原样" 提供以及任何明示或暗示的保证, 包括但不限于对适销性和特定用途适用性的默示保证被否认。在任何情况下, 版权所有者或贡献者均不得对任何直接的、间接的、偶然的、特殊的、惩戒性的, 或间接损害 (包括但不限于采购替代商品或服务;使用、数据或利润损失;或业务中断) 不论是造成什么原因的, 并且基于任何责任理论, 无论是在合同、严格责任或侵权行为 (包括疏忽或其他原因) 以任何方式产生于本软件的使用, 即使被告知这种损害的可能性。

## 第三方文件和许可证

Redis 使用来自第三方的源代码。所有这些代码都包含 BSD 或 BSD 兼容许可证。以下是第三方文件及其版权信息的列表。

*   Redis 使用[LHF 压缩库](http://oldhome.schmorp.de/marc/liblzf.html). LibLZF的 版权是 Marc Alexander Lehmann, 并根据 **两条款 BSD 许可证**.

*   Redis 使用 `sha1.c` 文件由 史蒂夫·里德 版权所有, 并在 **public domain**. 该文件非常受欢迎, 并在开源和专有代码中使用。

*   在 Linux 上编译时, Redis 使用[jemalloc](https://github.com/jemalloc/jemalloc), 由 Jason Evans, Mozilla 基金会 和 Facebook, Inc.版权所有, 并在 **两条款 BSD 许可证**.

*   在 Jemalloc 内部, 文件 `pprof` 受 Google Inc. 的版权保护, 并在 **三条款 BSD 许可证**.

*   在 Jemalloc 内部的文件 `inttypes.h`,`stdbool.h`,`stdint.h`,`strings.h`在`msvc_compat`目录是版权 亚历山大·切梅里斯和发布在**三条款 BSD 许可证**.

*   **hiredis**和**linenoise** 两个库也包含在 Redis 发行版中, 还包括 Salvatore Sanfilippo 和 Pieter Noordhuis 的版权, 并分别根据**三条款 BSD 许可证**和**两条款 BSD 许可证**.
