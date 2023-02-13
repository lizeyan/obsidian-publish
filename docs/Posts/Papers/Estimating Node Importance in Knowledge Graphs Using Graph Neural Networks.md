---
share: true
tags:
  - Paper Notes
  - KDD
  - "2019"
---

## Introduction

![Estimating Node Importance in Knowledge Graphs Using Graph Neural Networks_image_1](../../attachments/Estimating%20Node%20Importance%20in%20Knowledge%20Graphs%20Using%20Graph%20Neural%20Networks_image_1.png)

centrality 是在说结果和节点的connectivity有关。但是这不是知道网络结构自然就知道了么？

flexibility，input score不限定于特定的类型，但是别的方法不应该的也不限定么？

## Methodology

![Estimating Node Importance in Knowledge Graphs Using Graph Neural Networks_image_2](../../attachments/Estimating%20Node%20Importance%20in%20Knowledge%20Graphs%20Using%20Graph%20Neural%20Networks_image_2.png)

Objective：

![Estimating Node Importance in Knowledge Graphs Using Graph Neural Networks_image_3](../../attachments/Estimating%20Node%20Importance%20in%20Knowledge%20Graphs%20Using%20Graph%20Neural%20Networks_image_3.png)

所以没有signal给到的节点对objective是没有直接影响，只能通过和其他节点的连边去影响。

## Evaluation

除了PR，PPR，HAR这些之外，还和一众传统的的有监督方法进行了对比 （LR，RF，MLP，以及一个基础的GNN）

## Thinking and Conclusion