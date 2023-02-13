---
share: true
tags:
  - Paper Notes
  - ICCV
  - "2017"
---


传统的聚类方法依赖预先定义的距离函数. 基于深度学习的方法, 往往采用学习representation然后继续使用传统聚类方法的框架. 但是这种多步骤的框架在实践中过于笨重, 而且这些representation缺乏针对聚类进一步调整的空间.

这篇文章将聚类问题看成binary classification, 即每一对样本是否属于同一个类别(1或0). 由于实际上聚类是无监督的, 我们没有ground truth, 所以这篇文章提出了一个adaptive clustering算法.

![Untitled](../../attachments/Untitled.png)

DAC对问题的建模如下式

$$
\min_{\mathbf{w}}E(\mathbf{w})=\sum_{i, j}L(r_{ij}, g(\mathbf{x_i}, \mathbf{x_j}|\mathbf{w}))
$$

其中$r_{ij}$是样本i和j的相似度label, 是一个binary变量. 它的实际值是未知的.

$g(\mathbf{x_i}, \mathbf{x_j}|\mathbf{w})$是模型得到的esitimated similarity, $\mathbf{w}$是模型的参数.

$L$是binary cross entropy loss.

在DAC中, estimated similarity是通过label features得到的. 对于每个样本$\mathbf{x_i}$, 模型会输出一个label feature $\mathbf{l}_i\in \mathbb{R}^{k}$, k是类别的个数. 那么$g(\mathbf{x_i}, \mathbf{x_j}|\mathbf{w})=Cosine(\mathbf{l}_i, \mathbf{l}_j)$.

DAC进一步限制lable features为所有元素非负的单位向量, 即$||\mathbf{l}_i||_2=1, \mathbf{l}_{ih}\ge 0$

所以$g(\mathbf{x_i}, \mathbf{x_j}|\mathbf{w})=\mathbf{l}_i\cdot \mathbf{l}_j$

如果$E(\mathbf{w})$被最小化, 即$\forall i, r_{ij}=\mathbf{l}_i\cdot \mathbf{l}_j$, 那么可以证明$\mathbf{l}_i$是表示$\mathbf{x}_i$类别的one-hot vector.

既然没有$r_{ij}$, DAC就需要生成标注, 生成的方法是选择cosine similarity够大的样本对. 即

$$
r_{ij}=\begin{cases}1 & \mathbf{l_i}\cdot\mathbf{l}_j\ge u(\lambda)\\0 & \mathbf{l_i}\cdot\mathbf{l}_j \le l(\lambda) \\ None & otherwise\end{cases}
$$

其中$\lambda$是控制阈值变化的参数.

用label feature生成标注, 再用标注训练网络输出lable faeture, 这看起来是一个循环论证的问题. DAC对此提出的解释是, 对于一个随机初始化的ConvNet, 其中的filter也可以提出一些low-level feature, 因此此时label feature相似度很高的样本依然很可能是相似的样本. DAC的模型用的是ConvNet.

在训练过程中, 会保证$u(\lambda)\propto \lambda$, $l(\lambda)\propto-\lambda$

最终DAC的模型可以被写成下图的形式

![Untitled](../../attachments/Untitled 1.png)

DAC在优化时对$\mathbf{w}$和$\lambda$是分别优化的.

在每一个minibatch, DAC基于固定的$\lambda$计算$r_{ij}$和$v_{ij}$,然后更新$w$. 在每一个epoch结束后, 更新一次$\lambda$(固定w). 注意这里$\lambda$的梯度是常数,并不需要计算.

对feature vector的限制是通过在输出层做normalize实现的,并不需要加惩罚项