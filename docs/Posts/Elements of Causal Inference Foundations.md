---
share: true
tags:
  - Book Notes
  - Causal Inference
---

## Chapter 2: Cause-Effect Models
### SCM 的记号
假设有如下这样一个 SCM
$$
\begin{aligned}
C &:= N_C \\
E &:= f_E(C, N_E)
\end{aligned}
$$
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
$$
\begin{aligned}
C := N_C \\ E := 4\cdot C + N_E \\ N_C, N_E\stackrel{i.i.d.}{\sim}\mathcal{N}(0, 1)
\end{aligned}
$$
Show that $P_{C|E=2}$ follows Gaussian distribution.
**Solution**:

$$
p_{C|E=2}=\frac{p(E=2|C)p(C)}{p(E=2)}
$$