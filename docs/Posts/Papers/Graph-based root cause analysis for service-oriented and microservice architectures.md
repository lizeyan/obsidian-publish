---
share: true
tags:
  - Paper Notes
  - JSS
  - "2020"
---


## Introduction

针对service-oriented architecture (e.g. microservice architecture)，提出了一个RCA算法。解决的挑战是如何基于负载的依赖关系找到根因，从而利用现有的工具的自愈功能实现自动修复（例如kubernetes可以自动重启有问题的容器以及把容器移动到可以正常运行的宿主机上）。 

![Graph-based root cause analysis for service-oriented and microservice architectures_image_1](../../attachments/Graph-based%20root%20cause%20analysis%20for%20service-oriented%20and%20microservice%20architectures_image_1.png)

方法的逻辑是1. 提出了一个通过图表示系统整体状态的方法 2. 实现了一个图对比的引擎，可以和历史上的故障单进行比较，从而得到根因。

为了减少图比较的开销，可以加入异常检测并且只比较异常区域（异常节点附近的子图）。

## Methodology

## Evaluation

在三种不同的场景下进行了评估：HAproxy+HTTP 服务器，Kafka，Spark+HDFS

注入了几种故障：stress-ng on host，tc，wrong balancing（错误的HAproxy配置），stressing endpoint

## Thinking and Conclusion