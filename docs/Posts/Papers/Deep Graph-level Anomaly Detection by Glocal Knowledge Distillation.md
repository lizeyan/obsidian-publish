---
share: true
tags:
  - Paper Notes
  - WSDM
  - "2022"
---


Graph-level anomaly detection (GAD) (detecting abnormal graphs) 问题指的是给定一个有很多图的数据集, 找到其中和整个数据集大多数图显著不同的哪些图. 这个问题的实际场景包括识别药物的副作用, 识别有毒的化合物等, 或者可以用于调用链/交易线等的异常检测.

GAD包含两类异常, locally-anomalous graph指的是图上的个别节点及其邻居有异常的节点特征或者结构, gloally-anomalous graph指的是整个图的特征有异常.

和被研究的比较多的graph stream anomaly detection 不同. Graph stream是一个随时间变化的图的流, 其中的相邻时间的图大部分的节点和边都是不变的. 

现有的深度学习方法用于图上异常检测的方法主要是AE或者GAN的模式, 但这个方法大多是用于节点或者边的异常检测.

将这类方法用于GAD的一个挑战是很难从embedding忠实地重构一个图.

这篇文章提出了一个基于random knowledge distillation的GAD方法, GLocalKD. 它可以同时检测local和global anomalous graph.

![Untitled](../../attachments/Untitled.png)

GLocalKD首先准备两个GNN网络. 其中Random Target Network的权重固定在随机初始化的状态. 另一个网络是可训练的. 然后训练目标是分别比较两个网络的node representation和graph representation, 并把local和global的loss相加. 最终目标是让predictor network学习到random target network的输出. 最后的异常检测指标是predictor network和random target network输出的node和graph repsentation的距离.

这里面的key intuition是, 如果一个模式频繁地出现在random target network的latent space中, 那么这些模式就会被distille得更好. 

*我理解这里的intuition和用AE做异常检测是很相似的, 都是对原始数据进行降维, 然后利用降维过程只能保留主要特征的特点进行异常检测. 只不过, AE学习到的降维的目标是能够重构会原始数据, 但是在graph上做重构很难, 所以把降维的目标固定成一个随机的映射, 而不追求更难的重构了. 这个模型能work, 说明只是为了做异常检测的话, AE这种做到忠实地重构原始数据的是overqualified了, 并不需要重构.*