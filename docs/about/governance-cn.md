---
title: "redis 开源管理"
linkTitle: "Governance"
weight: 3
description: >
    开源项目的管理模式
aliases:
    - /topics/governance
---


从 2009 年到 2020 年，Salvatore Sanfilippo 构建，领导和维护 Redis 开源项目。在此期间，Redis 没有正式的治理结构，主要作为[迭代](https://en.wikipedia.org/wiki/Benevolent_dictator_for_life)-样式项目。

随着 Redis 的成长、成熟和用户群的扩大，为其持续开发和维护形成一个可持续的结构变得越来越重要。Salvatore 和 Redis 的核心贡献者希望确保项目的连续性并反映其更大的社区。 考虑到这一点，采用了新的治理结构。

## 目前的治理结构

从 2020 年 6 月 30 日开始，Redis 采用了 *轻治理* 模型，与项目的当前大小匹配，并最大限度地减少与其早期模型相比的更改。治理模式旨在成为一种精英管理，旨在赋予表现出长期承诺并做出重大贡献的个人权力。

## Redis 核心团队

萨尔瓦多·桑菲利波（Salvatore Sanfilippo）任命了两位继任者来接管并领导 Redis 项目：Yossi Gottlieb（[约西戈](https://github.com/yossigo)）和奥兰·阿格拉（[奥拉那格拉](https://github.com/oranagra))

在 Redis Ltd. 的支持下和祝福下，我们借此机会创建了一个更加开放，可扩展和社区驱动的 "核心团队" 结构来运行该项目。核心团队由根据长期个人参与和贡献选出的成员组成。

目前的核心团队成员是：

*   项目负责人：Yossi Gottlieb （[约西戈](https://github.com/yossigo)）来自 Redis Ltd.
*   项目负责人：奥兰·阿格拉（[奥拉那格拉](https://github.com/oranagra)）来自 Redis Ltd.
*   社区负责人：Itamar Haber （[itamarhaber](https://github.com/itamarhaber)）来自 Redis Ltd.
*   成员：赵昭（[索洛雷斯托伊](https://github.com/soloestoy)） 来自阿里巴巴
*   成员： 玛德琳·奥尔森 （[马多尔森](https://github.com/madolson)） 来自 Amazon Web Services

Redis 核心团队成员为 Redis 开源项目和社区提供服务。他们应该按照采用的行为，文化和语气树立一个良好的榜样。[行为准则](https://www.contributor-covenant.org/).他们还应该以一种不受外国或冲突利益的方式考虑项目和社区的最佳利益并采取行动。

核心团队将负责 Redis 核心项目，该项目是 Redis 的一部分，托管在 Redis 主存储库中，并已获得 BSD 许可。它还将致力于与构成 Redis 生态系统的其他项目保持协调和协作，包括 Redis 客户端，周边项目，依赖 Redis 的主要中间件等。

#### 角色和职责

核心团队的职责如下：

*   管理核心 Redis 代码和文档
*   管理新的 Redis 版本
*   保持高水平的技术方向/路线图
*   提供快速响应，包括修复/补丁，以解决安全漏洞和其他主要问题
*   项目管理决策和变更
*   Redis 核心与 Redis 生态系统其余部分的协调
*   管理核心团队的成员

核心团队旨在通过进一步将任务委派给表现出承诺，专业知识和技能的个人来形成并赋予贡献者社区权力。特别是，我们希望看到社区在以下领域有更多的参与：

*   报告问题的支持、故障排除和错误修复
*   贡献/拉取请求的分类

#### 决策

*   **正常决策** 将由核心团队成员基于懒惰的共识方法进行：每个成员可以投票 +1（正面）或 -1（负面）。反对票必须包括彻底的推理，更好的是，还有替代提案。核心团队将始终试图达成完全共识，而不是多数共识。正常决策示例：
    *   拉取请求和关闭问题的日常审批
    *   开放新问题供讨论
*   **重大决定** 对 Redis 架构、设计或理念以及核心团队结构或成员变更产生重大影响的内容最好通过完全共识来确定。如果团队无法达成完全共识，则需要多数票。重大决策示例：
    *   对 Redis 核心的根本性更改
    *   添加新的数据结构
    *   创建新版本的 RESP（Redis 序列化协议）
    *   影响向后兼容性的更改
    *   添加或更改核心团队成员
*   项目负责人有权否决重大决策

#### 核心团队成员

*   核心团队预计不会终身服务，但是，需要长期参与，以便在 Redis 编程风格和社区中提供稳定性和一致性。
*   如果必须替换由 Redis Ltd. 资助的核心团队成员，则在与其余核心团队成员协商后，Redis Ltd. 将指定替换成员。
*   如果非 Redis Ltd. 资助的核心团队成员将不再参与，无论出于何种原因，其他团队成员将选择替代者。

## 社区论坛和交流

我们希望 Redis 社区尽可能热情和包容。为此，我们通过了[行为准则](https://www.contributor-covenant.org/)我们要求所有社区成员阅读和观察。

我们鼓励所有重要的通信都是公开的、异步的、存档的和开放的，以便社区积极参与使用所述渠道。[这里](https://redis.io/community).例外情况是敏感的安全问题，需要在公开披露之前解决。

要就不当行为或安全问题等敏感事项与核心团队联系，请发送电子邮件至<redis@redis.io>.

## 新的 Redis 存储库和提交审批流程

Redis 核心源存储库托管在<https://github.com/redis/redis>.我们的目标是最终在 Redis GitHub 组织下托管所有内容（Redis核心源和其他生态系统项目）<https://github.com/redis>).提交到 Redis 源码库需要代码审查，至少需要一个不是提交作者的核心团队成员的批准，并且没有异议。

## 项目和开发更新

与项目和社区保持联系！有关项目和社区更新，请关注项目[社区](https://redis.io/community).开发公告将通过以下方式发布[Redis 邮件列表](https://groups.google.com/forum/#!forum/redis-db).

## 对这些治理规则的更新

对这些规则的任何实质性更改都将被视为重大决定。微小的修改或部长级更正将被视为正常决定。
