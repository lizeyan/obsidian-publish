---
share: true
tags:
  - Paper Notes
  - Information Systems Frontiers
  - "2020"
---

![LogGAN a Log-level Generative Adversarial Network for Anomaly Detection using Permutation Event Modeling_image_1](../../attachments/LogGAN%20a%20Log-level%20Generative%20Adversarial%20Network%20for%20Anomaly%20Detection%20using%20Permutation%20Event%20Modeling_image_1.png)
首先有个模板解析器，把非结构化的日志转化成模板

然后训练一个conditional generative model，给定一个时序的模板元组(c1, c2, c3, c4)，模型学习的是P(c4|c1, c2, c3)，用来做异常检测。这个文章用的是LSTM，和GAN作为生成式模型

为了解决样本不平衡的问题，用了一个permutation的技术生成新的样本

但是这个里面GAN的用法和一般的并不一样，一般生成式模型生成的是样本，而不是样本的概率分布。它这么做的原因也无非是因为用GAN是没有办法简单地得到概率密度的，只能去逐个生成样本，没法用来异常检测（当然也有一些工作在解决这个问题）。但是这么做的话，这个GAN的意义在哪里？