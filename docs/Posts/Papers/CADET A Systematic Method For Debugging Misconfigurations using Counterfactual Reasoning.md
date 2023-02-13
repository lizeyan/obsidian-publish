---
share: true
tags:
  - Paper Notes
  - arXiv
  - "2020"
---


## Introduction

misconfiguration可能会导致non-functional faults (比如latency下降)。由于现代的software的configuration非常多，导致定位misconfiguration非常困难

现有的方法通过传统的，或者ML-based的方法去寻找一个全局（几乎）最优的配置，但是这些方法缺乏对how和why的回答。没有挖掘内在的因果关系，而只是分析configuration和non-functional property之间的correlation可能会带来风险。

例如，从下图（a）来看，GPU growth（*不太懂但是好像是显存的某种限制*）增长会导致latency增加，但是这个结论是反直觉的。如果我们引入别的因素，swap memory来看，就会发现gpu growth增加和swap memory的增加是正相关的，而swap memory增加和latency增加是正相关的。但是如果限制swap memory不变，gpu growth增加，那么gpu growth和latency是负相关的。

![CADET A Systematic Method For Debugging Misconfigurations using Counterfactual Reasoning_image_1](../../attachments/CADET%20A%20Systematic%20Method%20For%20Debugging%20Misconfigurations%20using%20Counterfactual%20Reasoning_image_1.png)

![CADET A Systematic Method For Debugging Misconfigurations using Counterfactual Reasoning_image_2](../../attachments/CADET%20A%20Systematic%20Method%20For%20Debugging%20Misconfigurations%20using%20Counterfactual%20Reasoning_image_2.png)

因此这篇文章（CADET）采用了causality mining的方法。首先挖掘一个causality graph，然后对其上的causal path进行排序，找到对non-functional property （例如latency）影响最大的路径，从而指导configuration优化。

## Methodology

###### 1 causal graph discovery

causal graph是一个 acyclic directed mixed graph （ADMG）。图中包含的节点有三种

1. 配置项
2. non-functional property （例如 latency）
3. system events，例如系统负载。这类节点是不可变的。只能取observation的值

CADET采用了fast causal inference （FCI）算法挖掘因果关系，通过Fisher‘s exact test检验条件独立性。

> We picked FCI because it accommodates for the existence of unobserved confounders
> 

(不过我还不清楚FCI和PC有什么区别)

基于FCI可以得到一个无向图，然后基于edge orientation rules判断边的方向。最后把未判断方向的部分和NOTEARS’ DAG进行比较得到方向（*？？？*）

###### 2 causal path extraction

causal path通过从non-functional 节点向后遍历到源节点就能找出来

对causal path进行开排序的指标是average causal effect (ACE)

$$
ACE(Z, X)=\frac{1}{|X|}\sum_{a,b\in X} (\mathbb{E}[Z|do(X=a)]-\mathbb{E}[Z|do(X=b)])
$$

ACE的核心idea就是看变量X对Z的平均的causal effect。*但是问题是do-calculus （$P(Z|do(X=a))$）怎么算？*

ACE最大的X就是优化Z最优先需要考虑的项

但是具体如何优化呢？

记一个优化操作是$r$，那么这个操作的有效性就是ITE （individual treatment effect）

![CADET A Systematic Method For Debugging Misconfigurations using Counterfactual Reasoning_image_3](../../attachments/CADET%20A%20Systematic%20Method%20For%20Debugging%20Misconfigurations%20using%20Counterfactual%20Reasoning_image_3.png)

CADET会枚举所有的$r$，然后计算使得ITE最大的$r$。

## Evaluation

数据的收集方法是运行并监控一个软件系统，人工调优configuration。

数据集中包含5个软件系统，运行在3中硬件平台上（15种组合）。配置项有28项，10项软件的，8项操作系统的，10项硬件的（*都不是很多*）。

![CADET A Systematic Method For Debugging Misconfigurations using Counterfactual Reasoning_image_4](../../attachments/CADET%20A%20Systematic%20Method%20For%20Debugging%20Misconfigurations%20using%20Counterfactual%20Reasoning_image_4.png)

## Thinking and Conclusion

想法是很好，但是里面的细节是如何做的很多都不清楚。