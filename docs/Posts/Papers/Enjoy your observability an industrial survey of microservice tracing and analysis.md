---
share: true
tags:
  - Paper Notes
  - Empirical Software Engineering
  - "2021"
---

这篇文章通过访谈的形式对现在业界的tracing system的使用情况做了调研.

在Section 3中, 主要介绍了研究的全过程. 包括以下步骤

1. 选择访谈对象
2. 对访谈对象分组
3. 准备访谈问题模板
4. 进行pilot interview, 根据interviewee的反馈调整访谈流程
5. 访谈.
6. 后续分析

这篇文章访谈涉及了10个微服务系统, 大部分都有tracing system, 只有一个特别小的ERP系统不准备使用tracing system.

在记录日志的方式中, 绝大多数系统都采用的是使用tracing framework的方法, 使用dynamic binary instrument的很少. 有一些采用manually code的方法. 小型系统使用这种方法可以避免使用比较重的tracing system. 一些大型项目在tracing framework之外也使用来MC, 主要目的是为了实现一些特别的需求.

大部分的项目都使用了自定义的日志格式而不是opentracing标准, 一方面是因为又的项目开始较早, 另一方面是有些项目有特殊需求. 但是很多项目都有迁移到opentracing的计划, 因为基于opentracing的开源社区非常活跃.

一些大型项目会进行head-based sampling以减少开销, 小型项目基本不需要.

大型项目普遍使用HBase等数据库进行存储以获得最佳的效率, 但是小型项目可以使用ELK等工具方便实现.