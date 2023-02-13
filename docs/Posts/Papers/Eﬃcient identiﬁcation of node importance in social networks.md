---
share: true
tags:
  - Paper Notes
  - Information Processing and Management
  - "2016"
---



## Introduction

节点重要性排序

大概方法是提出了5个计算节点重要性的基础方法，然后用了一套规则去综合考虑这5个指标。但是intuition不明确，evaluation不充分。没什么参考意义。

## Methodology

structural hole 节点i是不是尽可能多地直接连接了其他节点（而且通过其他节点中转的路径尽可能少）

flow betweenness 当任意两个节点作为s和t做最大网络流的时候，通过i传递的流的比例之和

culmulative nomination 类似page rank

information indices

subgraph centrality 

## Evaluation

## Thinking and Conclusion