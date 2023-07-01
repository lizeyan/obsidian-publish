---
share: true
tags:
  - Paper Notes
  - WWW
  - "2020"
---


## Introduction

1. 建立Anomaly Behavior Graph （和之前的工作基本没什么差别）
    1. 依据节点之间的相关性，每个指标分别计算
2. update metric weight
    1. 根据historical similar anomaly profile计算weight
3. random walk
    1. 和之前的工作没什么差别

边的方向的确定，通过v structure

![https://lizeyan-ipic.oss-cn-beijing.aliyuncs.com/image-20200323195814858.png](https://lizeyan-ipic.oss-cn-beijing.aliyuncs.com/image-20200323195814858.png)

$v_i$和$v_l$给定$v_j$时是条件独立的，所以自然均应该指向$v_j$

剩下的规则都是为了维护v structure或者无环

![https://lizeyan-ipic.oss-cn-beijing.aliyuncs.com/image-20200323201208283.png](https://lizeyan-ipic.oss-cn-beijing.aliyuncs.com/image-20200323201208283.png)


## Methodology

## Evaluation

## Thinking and Conclusion