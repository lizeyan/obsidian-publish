---
share: true
tags:
  - Paper Notes
  - CIKM
  - 2021
---

这篇文章没有太多技术性内容, 整体上是阿里云对自身使用的一套RCA框架的概述.

在大规模云计算的根因定位问题中, 有几个主要的挑战.

1. 不断增加的复杂性和规模, 以及随之而来的多类型, 大量的监控数据
2. 实时性的要求
3. 实际系统中数据是稀少的
4. 阿里云中有三种不同的计算架构

而现有的根因定位工作 (e.g., CloudRanger, OM knowledge)有两个主要的局限

1. 首先没有同时利用指标和日志数据
2. 其次定位结果往往需要进一步人工排查, 不能直接开始take action

这篇文章提出了一个新的RCA workflow叫做CloudRCA.

![Untitled](../../attachments/Untitled.png)

首先, 对于指标数据, CloudRCA首先采用RobustSTL进行时间序列分解. 然后通过对每个部分进行特定的statistical test来检测特定类型的异常. 这些异常被转化成新的时间序列, anomaly sequence.

对于日志数据, CloudRCA首先使用AFT-tree (FT-tree的一个改进)提取模板, 然后用聚类方法进行聚类. 每个类就是一个log pattern. Log pattern的数量被转化为新的时间序列.

对于这些metrics (包括anomaly sequence和log pattern squences), 为了去掉对RCA没有帮助的, 通过TF-IDF来筛选. *这一块没有细说, 推测大概是计算在和故障发生关联比较大的指标. 另一块就是文字里似乎说的是两类指标都会用TF-IDF筛选, 但是图中只画了log pattern sequence.*

最后会利用这些指标计算一个Bayesian network. 指标之间的结构是由PC算法计算的, 其他的实体的结构是由CMDB指定的.然后通过最大似然的方法计算这个BN上的条件概率表. *这里不是很清楚为什么要这么设计BN的结构, 为什么root cause layer在中间. 下面的type说的是什么东西也不太清楚.*

![Untitled](../../attachments/Untitled 1.png)

当故障发生时, 把条件概率最大的root cause layer中的节点作为根因.