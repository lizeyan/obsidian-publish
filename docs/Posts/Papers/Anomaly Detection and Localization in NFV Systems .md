---
share: true
tags:
  - Paper Notes
  - NOMS
  - "2022"
---


这篇文章提出了一种对NFV（network function virtualization）系统的异常检测和定位方法。异常检测指的是，判断在时刻t一个系统是不是异常，定位指的是判断异常发生在哪个VNF （virtual network function）。这里的定位并没有考虑故障传播，更类似对异常检测的解释。

![Untitled](../../attachments/Untitled.png)

一个NFV系统包含若干个VNF，VNF之间需要互相调用来实现功能。不过NFV中的VNF看起来数量较少，而且每个VNF都是异质的。

本文的方法需要的输入数据是每个VNF上的时序指标。不同VNF上的指标的类型和数目都是不同的。输出包括时刻t系统是or不是异常，如果是异常的话，哪个VNF是异常的位置。

这篇文章的创新点主要有几个部分。

首先，由于异常标注的缺乏，异常检测方法往往需要无监督方法，这些无监督方法假设训练数据中都是正常的。但是无监督方法会受到训练数据中的contamination影响（不可避免）。因此，这篇文章提出使用noisy-student 方法减少contamination的影响。

这篇文章的Nosiy-student方法的流程如下。

首先，采用一个无监督的异常检测方法（这篇文章采用了DAGMM），然后找到置信度比较高的正常和异常样本。

通过这些pseudo-labled sample，训练一个deviation network，来给出每个样本的异常分数。

为了进一步提高deviation network的性能，这篇文章在样本和deviation network之间接了一个auto encoder，输入到deviation network的数据是$[\text{encoer}(x):\text{rec}(x)]$。

训练的loss分成两部分，对于正常样本（pseudo label），同时优化auto encoder的重构误差和deviation network给出的异常分数。对于异常样本（pseudo label），只优化异常分数。但是设立一个上界，异常分数超过这个上界之后loss就是0.

![Untitled](../../attachments/Untitled 1.png)

这篇文章的第二个创新点是它的定位方法。这篇文章提出了一个类似explaination power的解释方法。它将被检测为异常的样本的每一个VNF的指标，替换成最相似的正常样本的对应指标，然后检查异常分数的变化。如果异常分数的最大变化超过阈值，那么异常位置就是最大变化对应的VNF，否则就找不到异常位置。在找不到异常位置的时候，这篇文章采用了一个通用的局部解释方法，SHAP去分析每个feature的贡献，然后把平均贡献最大的VNF作为异常位置。