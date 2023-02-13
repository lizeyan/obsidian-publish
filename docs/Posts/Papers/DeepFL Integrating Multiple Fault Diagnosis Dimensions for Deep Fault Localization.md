---
share: true
tags:
  - Paper Notes
  - ISSTA
  - "2019"
---


## Introduction

ISSTA'19 best paper

程序故障定位（指的是分析代码，找到故障的代码片段）很重要，有很多相关的研究。

SBFL轻量，高校。它通过对单元测试的统计分析给出定位结果，基本的思想是故障代码会让测试失败。然而问题是失败的测试执行的代码不全是让其失败的部分，而故障代码也可能会被成功的测试执行到。

MBFL基于对代码的变换（mutation）以及变换引起的测试结果的变化，从而分析代码段对故障的影响。但是问题是有的代码段的mutation难以定义。因此在一些情况下MBFL效果也很不好。

由于传统方法没法适应所有的情况，因此研究者开始通过机器学习综合不同的传统方法。

Learning-to-rank（好像说的只是线性回归而已啊...）在很多方法中被采用并证明了其效果，但是learning-to-rank不能学习到数据中关键的feature。

DeepFL使用深度学习解决这个问题，使用SBFL，MBFL，以及复杂度分析，文本相似度比较等传统方法作为feature，提出了一个MLP_DFL架构。DeepFL发现MBFL是最重要的feature。

## Methodology

###### 基本架构

作者claim不同来源的feature有不同的特性，有必要分开考虑。

作者先介绍了利用RNN做这个的思路：

![Untitled](../../attachments/Untitled.png)

但是RNN架构的缺点就是不同的group是共享权重的，并且无法引入更复杂的设计

因此作者提出了这样的多层次的MLP的设计： 

![Untitled 1](../../attachments/Untitled%201.png)

###### Features

SBFL 34个

MBFL 140个

statistical code metric 21（源代码） + 37 （字节码）

textual similarity 用代码和failed test的field（分别有5个（方法名，类名，变量，注释，调用的方法）和3个（测试名，代码，失败信息），组合起来就是15个）进行文档搜索，用TF-IDF作为feature。

一个training instance是一个代码段（statement，block，method等）

###### Loss Function

pairwise learning to rank的loss或者cross entropy

## Evaluation

数据集，公开数据集

RQ1: 定位性能是不是达到了state-of-the-art。是的。用的是留一验证，验证一个故障的时候，把其他所有故障作为训练集。

RQ2: DeepFL的不同组件和feature对结果的影响。对比了不同的网络结构，不同feature

RQ3: 训练epoch和loss function如何影响结果。

RQ4: 迁移性如何，能不能在其他Project上工作？可以。做了两个模式的对比，验证在某个project的性能时，用其他所有project的数据做训练集；把故障打乱，用10-fold cross validation做验证。

## Thinking and Conclusion

为了解决传统方法只能针对特定场景的问题，DeepFL用机器学习方法（一个MLP的简单变种）综合了很多（100多个feature）传统方法。

当然DeepFL不是第一篇，但是DeepFL第一个提出了使用DL，之前的使用的是SVM这种传统机器学习模型。

从结果上来看，DeepFL的性能比SOTA的传统机器学习综合定位工作（用的是SVM，Adaboost这种）高了一些，DL是有用的。

但是文章一开始是说传统的learing-to-rank工作没有自动选择合适的feature或者选到更高级的feature（所以就需要人工选择feature），而DL的优点就是可以输入（相对）原始的数据，没有特征工程，相当于自动学到有用的feature。

性能的差距可以indicates DeepFL确实学到了什么潜在的内涵，但是到底是不是这样呢？也可以猜测是不是只是因为DL算法的capacity比其他算法大，或者DL比SVM之类的好调参数？这篇文章只是简单对比了一个四类feature哪一类对结果影响更大

有监督方法的最大的缺点就是标注数据不好获得。

DeepFL应对这个问题的方法是证明了他们的方法可以在一个project上训练，直接应用到其他project上。感觉这样已经足够了。

如果还需要进一步的话，1）可以说明训练集覆盖的类型对结果的影响变化曲线 2）可以说明fine tuning性能