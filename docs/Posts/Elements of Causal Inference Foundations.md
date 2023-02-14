---
share: true
tags:
  - Book Notes
  - Causal Inference
---

## Chapter 3: Cause-Effect Models
### SCM 的记号
假设有如下这样一个 SCM
\n$
\begin{aligned}
C &:= N_C \\
E &:= f_E(C, N_E)
\end{aligned}
\n$
所以如果$N_E$的值确定，为$n_E$时，$E=f_e(C, n_e)$是唯一确定的
如果记$C$和$E$的取值范围是$\mathcal{C}$ 和$\mathcal{E}$，那么可以将$N_E$的取值空间看作是$C\mapsto E$的函数，即$E=n_E(e)$

### Intervention和 Counterfactual 的不同
假设$C$和$E$都是二值的，即$\mathcal{E}=\mathcal{C}=\{0, 1\}$，那么$N_E$的取值范围是$\{\mathbf{0}, \mathbf{1}, ID, NOT\}$，即恒为 0，恒为 1，全等和取反。假设现在有两个不同的$P_{N_E}$，其中$P_{N_E}^1$有一半一半的概率是$\mathbf{0}$和$\mathbf{1}$，$P_{N_E}^2$有一半一半的概率是$ID$或者$NOT$。那么此时两者有相同的边缘概率，$P_{E|C=0}$和$P_{E|C=1}$，都是 50%概率的伯努利分布。而因为$C$是$E$的原因，所以$P_{E|do(C)}$和$P_{E|C}$总是恰好相等的。也就是说，不论做什么干预，都不影响$E$的分布。但是显然$P_{N_E}^1$和$P_{N_E}^2$的反事实是不一样的。对于前者，如果$C$取了不同的值，$E$的值不会变化，而后者则会变化。
*我理解这里的区别在于噪声项$N_E$是否已经确定了，在反事实中，$N_E$的分布已经不是先验的P_{N_E}了*
以上分析说明，两个不同的 SCM 可能导致相同的 interventional distribution。此外，两个不同的 SCM 还能导致相同的 counterfactual distribution。
对$E := f_E(C, N_E)$中的$N_E$进行重参数化显然不影响$E$的分布，所以假设有一个$N_E$取值范围上的一一映射$g$，我们定义$\tilde{N_E}=g(N_E)$，那么$E := f_E(C, N_E)=f_E(C, \tilde{N_E})$
### Problems
#### Problem 3.6
**Problem**:
Given that 
\n$
\begin{aligned}
C := N_C \\ E := 4\cdot C + N_E \\ N_C, N_E\stackrel{i.i.d.}{\sim}\mathcal{N}(0, 1)
\end{aligned}
\n$
Show that $P_{C|E=2}$ follows Gaussian distribution.
**Solution**:
根据贝叶斯公式
\n$
p_{C|E=2}=\frac{p(E=2|C)p(C)}{p(E=2)}
\n$
其中，
\n$
p(C=c)=\mathcal{N}(0, 1)=\frac{1}{\sqrt{2\pi}}\exp(-\frac{c^2}{2})
\n$
\n$
p(E=x|C=c)=\mathcal{N}(4c, 1)=\frac{1}{\sqrt{2\pi}}\exp(-\frac{(x-4c)^2}{2})
\n$
\n$
p(E=x)=\int_{-\infty}^{+\infty}p(E=x|C=c)p(C=c)dc=N(0, 17)
\n$
[[https://www.wolframalpha.com/input?i=integrate+1+%2F+sqrt%282+*+%5Cpi%29+*+exp+%28-1%2F2+*+%28x+-+4c%29%5E2%29+*+1+%2F+sqrt%282+*+%5Cpi%29+*+exp+%28-1%2F2+*+%28c%29%5E2%29%2C+c+from+-inf+to+inf]]
所以，
\n$
\begin{aligned}
p_{C|E=2}&=\frac{\frac{1}{\sqrt{2\pi}}\exp(-\frac{(2-4c)^2}{2}) \cdot \frac{1}{\sqrt{2\pi}}\exp(-\frac{c^2}{2})}{\frac{1}{\sqrt{34\pi}}\exp(-\frac{2^2}{34})} \\
&=\frac{1}{\sqrt{\frac{2}{17}\pi}}exp(-2+8c-8c^2-\frac{1}{2}c^2+\frac{2}{17}) \\
&=\frac{1}{\sqrt{\frac{1}{17}}\sqrt{2\pi}}exp(-\frac{1}{2}\frac{(c^2-2\cdot\frac{8}{17}c +\frac{8}{17}^2)}{\frac{1}{17}})\\
&=\mathcal{N}(\frac{8}{17},\frac{1}{17})
\end{aligned}
\n$
#### Problem 3.7
![Elements of Causal Inference Foundations_image_1](../attachments/Elements%20of%20Causal%20Inference%20Foundations_image_1.png)
do(Y=constant)，如果此时 X与Y不独立，那么是前者，否则是后者
#### Problem 3.8
![Elements of Causal Inference Foundations_image_2](../attachments/Elements%20of%20Causal%20Inference%20Foundations_image_2.png)
Solution:
a)
\n$
\begin{aligned}
\begin{cases}
\alpha = 2\gamma + 1 \\
\beta = 2 \cdot \delta \\
\gamma=2\cdot\alpha \\
\delta=2\cdot\beta +1
\end{cases} \Rightarrow
\begin{cases}
\alpha=4\alpha + 1\\
\beta=4\beta + 2
\end{cases}
\Rightarrow
\begin{cases}
\alpha=-\frac{1}{3}\\
\beta=-\frac{2}{3}\\
\gamma=-\frac{2}{3}\\
\delta=-\frac{1}{3}
\end{cases}
\end{aligned}
\n$
b)
\n$
\begin{aligned}
\begin{cases}
\alpha = \gamma + 1 \\
\beta =  \delta \\
\gamma=\alpha \\
\delta=\beta +1
\end{cases} \Rightarrow
\begin{cases}
\alpha=\alpha + 1\\
\beta=\beta + 1
\end{cases}
\Rightarrow
\text{No solution}
\end{aligned}
\n$
为了让SCM有解，可以设$N_X=aN$, $N_Y=bN$
则上述方程可以变成
\n$
\begin{aligned}
\begin{cases}
\alpha a + \beta b = \gamma a + \delta b + a \\
\gamma a + \delta b = \alpha a + \beta b + b
\end{cases}
\Rightarrow a + b = 0
\end{aligned}
\n$
所以，可以令$N_X\sim N(0, 1)$, $N_Y:=-N_X$, $X:=2N_X$, $Y=-N_Y$

## Chapter 4: Learning Cause-Effect Models
### Structure Identifiability
是否可以仅从联合分布$P(X, Y)$，区分 X和 Y 的因果关系（即 strcuture）？答案是否定的，不行。
> [!定理] 给定任何一个联合分布$P(X, Y)$, 都可以找到一个SCM，其中$Y:=f_Y(X, N_Y)$, $X \perp \!\!\! \perp Y$

所以，必须要做出一些假设，在这些假设成立的前提下才能实现 indentifiability
一般这样的假设有两类，一种是限制$f_E$ 在一个比较小的类中，另一种是限制$P(C)$和$P(E|C)$。
> [!argument] We only argue for the belief that if there is a simple function that ﬁts the data, it is more likely to also describe a causal relation

### Linear models with Non-Gaussian Additive Noise
> [!Assumption] LiNGAM: $E = \alpha C + N_E$

