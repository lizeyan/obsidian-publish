---
share: true
tags:
  - Paper Notes
  - ASPLOS
  - "2021"
---


## Introduction

微服务根因定位，需要Trace

## Methodology

首先，我们可以对一个trace的latency分解为如下的树结构

![Untitled.png](attachments/Sage%20/Untitled.png)

（这个模型jaeger就可以做到，但是银行客户那边没见过这么详细的数据）

然后，我们基于这个模型，可以构造这么一个贝叶斯模型

![Untitled 1.png](/Untitled%201.png)

![Untitled 2.png](/Untitled%202.png)

每个RPC的latency分解为service side， client side， request/response network delay。

client side latency由来回的网络延迟和service side latency决定，而来回的网络延迟由网络性能指标和隐变量决定

这里的隐变量是通过CVAE来学习的。具体的操作论文里没有细讲，然而这里也不是很显然

根因判断是通过替换每个metric节点的值为正常值（历史数据的median），如果通过CVAE reconstruct （原文说的是generate，但是我觉得应该是reconstruct）的QoS恢复正常了，那么这个metric就是根因。

根据具体导致问题的metric类型，Sage会自动采取对应的操作

## Evaluation

![Untitled 3.png](/Untitled%203.png)

通过stress的方式注入异常，Sage能更快使系统恢复

## Thinking and Conclusion

整个想法都还挺有意思的