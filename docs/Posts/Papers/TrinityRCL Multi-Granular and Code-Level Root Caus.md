
---
share: true
tags:
- Paper Notes
- TSE
- "2023"
---

## 概述

在微服务系统中，在多个层次定位故障非常重要，因为实际上的根因可能发生在多个不同的层次，例如服务、机器、代码等等。

低层次的故障定位，包括根因指标定位和故障代码定位，都是必需的，因为这两个之间没有必然的关联。

微服务系统中主要的监控数据包括指标、日志和调用轨迹三类，但是目前的方法很少有全部使用的。

本文提出的 TrinityRCL 可以利用全部三种监控数据，在服务、机器、指标和代码等多个层次同时故障定位。特别是代码层次的定位，是此前的工作（指使用监控数据进行故障定位的工作）中特别缺少的。

TrinityRCL 的框架和此前的基于随机游走的工作仍然是非常类似的。它首先构建一个”因果图“，这个因果图是异构的：图上的节点包括服务、机器、指标，（代码中的）故障（即日志中的 exception）等。然后在因果图上，还是执行 random walk 那一套，来对根因节点进行排序。

为了构建因果图（其实不能叫因果图了），本文使用了多种拓扑关系。

首先，调用轨迹被用来寻找**前端入口异常服务**相邻的 k 跳的有异常调用的服务，作为服务之间的关系。服务和机器之间的关系通过部署关系获得。指标的机器的关系也是从 CMDB 中获得。故障和机器的关系取决去发现故障的日志在哪台机器上。这些关系都是有向的：从调用方指向服务方，或者从内容指向机器。

随机游走的转移概率仍然是基于指标相似度的，但是本文对不同类型的节点之间相似度的计算做了一些分类讨论（没有太 get 到分类的具体理由，但是反正他们做了一些分类讨论…）

## 讨论

这篇文章提到它的一个重大贡献是同时利用了三种类型的监控数据，但是实际上，调用轨迹数据仅仅用来获取服务的异常率，日志用来获取不同故障出现的频率，都是被转换成了简单的时序指标，然后和其他指标放在一起使用，其实是没有深入分析调用轨迹和日志。

另一方面，这篇文章也缺少对这方面的消融实验，没有分析增加新的类型的监控数据在多大程度上改善了效果。

而且在实验中本身对比的 baseline 就比较局限，分析也很少。因此该方法的效果，以及各部分的贡献，都有些不清楚。

## 总结

This paper proposes TrinityRCL, a method for multi-granular and code-level root cause localization using multiple types of telemetry data in microservice systems. TrinityRCL builds a heterogeneous graph of services, machines, metrics, and exceptions, and uses random walk to rank root cause nodes. While the method is able to use all three types of monitoring data, it does not deeply analyze call traces and logs, and lacks ablation experiments and analysis of the contribution of each component.