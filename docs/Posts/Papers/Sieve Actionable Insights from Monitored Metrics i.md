---
share: true
tags:
  - Paper Notes
  - Middleware
  - "2017"
---


这篇文章试图从microservice系统的监控中提取actionable insights. 现有的监控工具提供了大量的监控指标, 但是这些监控指标并没有能够转化成有用的知识. 这篇文章就是探索如何做到这一点.

Sieve实现actionable insights的手段是对于一个microservice system的指标, 提取出一个由关键指标组成的dependency graph. Sieve包含两个关键模块. 

- 首先Sieve筛选出重要的指标, 而丢弃掉其他的指标. 因为监控系统采集到大量的指标, 处理这些指标需要很大的overhead, 但是指标之间往往有很强的相关关系, 不需要都去考虑.
- 然后Sieve检测这些重要指标的依赖关系.

Sieve的方法主要有三步.

首先是数据采集, Sieve需要采集指标和compoenent之间的调用关系组成的call graph.

然后是指标聚类, 每一类的centroid指标就是重要指标. Sieve具体采用的聚类方法是k-shape. K-shape是一个基于形状的聚类方法, 它提出使用SBD (shape based distance)作为衡量指标相似度的方法, 并且提出如何基于SBD去计算指标的平均值. K-shape的整体算法逻辑类似k-means. 计算出所有的类指标, 对于每一类, 其中和centroid最相近的指标(centroid并不是真实存在的指标而是计算出来的平均值)就是这一类的代表指标. Sieve在应用k-shape的过程中做了三点改动, 首先是用三次差值填补缺失值, 然后是用指标名字决定初始类别划分(原算法是随机), 最后是基于silhoutette value动态决定最佳的类别个数.

最后是找到dependency graph. Sieve对call graph上相邻的一对compoenent的每一组关键指标组合, 依次判断是否有granger causality.

Sieve的方法有什么用呢?

这篇文章提出了两个use case.

首先是用于autoscaling, 具体来说就是利用关键指标和dependency graph去设计scaling rules.

然后是root cause analysis, 具体方法是比较正常和异常时候的dependency graph, 差别集中的组件可能就是根因.

*这篇文章提到的actionable insights和我们提出的failure dependency graph (FDG) 有什么不同呢?* 

*Sieve中的dependency graph, 每一个节点是一个compoenent下个一个关键指标, 这个关键指标是作为一个类的中心, 也代表了一类指标. 不同的节点指标的边是granger cause关系. 同时有一个限制就是节点代表的component必须是有调用关系的.* 

*我们提出的failure dependency graph上面每一个节点也是一组指标, 节点之间基本上也需要component有调用关系(没有严格要求,但是现有的实践都是这么做的).* 

*主要的不同在于节点的物理含义不同. FDG中的节点代表了一个failure instance, 表示这里面的指标组合被判断为根因的时候对应了一种具体的故障场景, 运维人员可以直接采取对应的mitigation actions. Failure instances之间的指标是可以有重叠的. 但是Seive中的一个节点只代表一组互相之间非常相关的指标, 和mitigation action实际上并没有关联.*

除此之外, 边的物理含义也有微弱的差别. *FDG上的边表示failure instance之间的依赖关系, 但是并没有确定指granger causality, 而是主要基于CMDB部署关系去确定的.*