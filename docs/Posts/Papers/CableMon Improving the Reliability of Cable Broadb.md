---
share: true
tags:
  - Paper Notes
  - NSDI
  - "2020"
  - Time Series Anomaly Detection
---

## Introduction

cable network在美国大量存在，但是由于年久失修，经常发生问题

PNM是一种用于主动检测cable network的问题的系统。它能够允许ISP去定期采集cable modal（CM）上的metric，用来提前检测是否发生了故障。

PNM数据可以被看作一个multi-variate time series data

这个问题的核心挑战在于没有systemic expert knowledge或者ground truth dataset。

这篇文章假设tickets（工单）意味着故障。但是这其中会存在一些噪声，因此CableMon过滤了所有工单，只选取解决动作是排除工程师和描述中存在特定关键词的工单。CableMon的目标就是达到high ticket prediction accuracy和medium ticket coverage

为什么不追求high ticket coverage？因为ISP的处理故障的瓶颈资源是工程师的人力，只要保证派出去的工程师不被浪费（high precision）就足够了，就算达到了high recall（ticket coverage）也没有能力去都解决。

工单除了误报还有很多漏报。实际上就算有故障发生，大多数用户（93.9%）也并不会报障。

文章因此设计了一套thresholding selection方法去解决这个问题

文章全部的分析和实验都来自一个中等大小的ISP的8个月的实际数据。其中只分析了upstream（上行）的数据。数据中大约包含60K个不同的用户。

## Methodology

motivation：limitation of previous works

previous works使用阈值选择。但是1）阈值很难选（本文下面提出了一个新的基于ticket rate ratio的方法） 2）这些方法直接对instantaneous value应用阈值，导致波动很大（本文是对WMA，EWMA等统计指标计算阈值）

本文的一个核心idea是应用tickets数据作为是否有故障的hint，来帮助判断PNM数据是否有故障。

但是如果把这个作为一个有监督二分类问题来做显然不合适，因为ticket仅仅是一些正例。没有ticket的并不代表没有故障，因为绝大多数用户（93.9%）遇到故障也不去报障。此外正例中也存在一些噪声，因为报障的也不见得一定是网络真的出了故障（但这任何方法都面临一样的问题，包括CableMon）。

首先对每个CM的所有time series，CableMon提取了几个经典的features：

![Untitled.png](/Untitled.png)

其中有几个算法需要超参数，因此CableMon同时采用了多种可能的超参数。

现在CableMon对原始的PNM数据得到了很多个time series features

对其中每一个feature，CableMon需要选择一个阈值。

阈值的原则是最大化ticket rate ratio。即按阈值划分abnormal duration和normal duration之后，abnormal duration和normal duration和ticket rate的比值。

CableMon通过最终的ticket rate ratio选择了一些useful features

最终通过一个滑动窗口算法给出decision：如果一个滑动窗口中的异常点比例超过阈值，则ISP需要实际派出工程师排查。这里窗口的大小和异常点比例的阈值都是超参数，

## Evaluation

evaluation metrics:

1. ticket prediction accuracy
如果一个连续的异常点区间里有一个真的ticket，那么这是一个true prediction。
ticket prediction accuracy=## true prediction / ## all prediction
2. ticket coverage
true prediction中包含的ticket数量/ticket的总数量

![两个超参数对结果的影响。还是比较敏感的](../../attachments/Untitled 1.png)

两个超参数对结果的影响。还是比较敏感的

![](../../attachments/Untitled 2.png)
![](../../attachments/Untitled 3.png)
![](../../attachments/Untitled 4.png)

![baseline也太弱了，有监督分类器存在很多不可克服的缺陷，但是还是远远超过previous work了](../../attachments/Untitled 5.png)

baseline也太弱了，有监督分类器存在很多不可克服的缺陷，但是还是远远超过previous work了

## Thinking and Conclusion