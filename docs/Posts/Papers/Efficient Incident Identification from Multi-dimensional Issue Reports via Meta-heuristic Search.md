---
share: true
tags:
  - Paper Notes
  - "ESEC/FSE"
  - "2020"
---


## problem formulation

把和iDice一样的寻找effective combinations的问题通过meta-heuristic search解决

首先，这个定位问题被formulate

$max_{x\in \mathcal{X}} R(x)$

其中$x$表示维度组合，$\mathcal{X}$是所有维度组合的集合（和我们的定义基本一样（squeeze实际上还限制了维度组合只能是出现过的组合，而这里是直接取了幂集），只是严格地表达了一下）：

$\mathcal{X}=\{x\in 2^T|\forall T_i, |s\cap T_i|\le 1\}$

其中$T=\cup_{i}T_i, T_i={a_i}\times V_i$，其中$a_i$是第i个attribute，$V_i$是$a_i$的所有可能取值的集合

$R(x)$是目标函数，表示一个attribute combination是不是effective combination

$R(x)=p_a(x)\log\frac{p_a(x)}{p_b(x)}$

其中$p_{\cdot}(x)=\frac{V_{xt}}{V_{t}}$，表示该attribute combination对应的issue数量和总issue数量的比值

a和b分别代表change point之后和之前的时间段

$\log\frac{p_a(x)}{p_b(x)}$表示change point之后相比之前issue比例变化的比例，$p_a(x)$表示change point的issue比例的绝对值的大小

#### search method

定义operator函数

`operator(c: AttributeCombination, a: Attribute, v: Optional[AttributeValue])->AttributeCombination`

即operator是对一个给定的attribute combination进行操作，得到一个新的attribute combination，需要的参数是某个attribute，以及对应的attribute value（optional）。

operator定义了四个：

1. 添加一个(a, v)的tuple
2. 修改某个a对应的v
3. 替换一对(a, v)
4. 删除一对(a, v)

整体的搜索流程就是如下的:

```python
c = emptyset
EC = emptyset
while not terminate_condition():
   if R(c) > threshold:
       EC = EC | c
   c = operator(c, a, v)
```

所以核心是如何选择每一步的operator和a和v。

operator的选择是完全随机的

a和v的选择是通过信息熵$E$，选择信息熵最大的a和v

$p(v_i^k)=\frac{count(v_i^k)}{\sum_{v_i^j \in V_i}count(v_i^j)}$，其中$count$表示对应的attribute value对应的issue的数量

$E(v_i^k)=-p(v_i^k)\log p(v_i^k)$

$E(a_i)=\sum_{v_i^k\in V_i}E(v_i^k)$

所以整体的算法就如下所示

$c=\emptyset$

$EC = \emptyset$

`while not terminate_condition():`

`if` $R(c)\ge \delta_r$:

$EC \leftarrow EC \cup c$

`operator` ← a random operator

`with probability` $p$:

$a$ ← $argmax_{a\in A} E(a)$

`if` `operator` `need attribute values:`

$v$ ← $argmax_{v\in V_a} E(v)$

`else:`

$v$ ← null

`else`:

$a$ ← a random attribute feasible for `operator`

`if` `operator` `need attribute values:`

$v$ ← a random attribute value of $a$ feasible for `operator`

`else:`

$v$ ← null

$c$ ← `operator`($c$, $a$, $v$)