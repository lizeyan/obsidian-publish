---
share: true
tags:
---

|Name|Graph Structure|Multiple Predicates|Node Features|Signal (Signal)|Signal (Multi)|备注|
|---|---|---|---|---|---|---|
|PR|✅|❌|❌|❌|❌||
|PPR|✅|❌|❌|✅|❌||
|HAR|✅|✅||✅|❌||
|GENII|✅|✅|✅|✅|❌||
|MultiIImport|✅|✅|✅|✅|✅|
|HITS|✅|❌|❌|❌|❌||
|IPM2016|✅|❌|❌|❌|❌|自定义的多signal|

## Random Walk

### PageRank (PR)

$$
\mathbf{r}^\top=\mathbf{r}^\top((1-\alpha)\mathbf{W} + \alpha \mathbf{1} \mathbf{u}^\top)
$$

$$
\mathbf{u}=\frac{1}{N}\mathbf{1}
$$

### Personalized PageRank (PPR)

$$
\mathbf{r}^\top=\mathbf{r}^\top((1-\alpha)\mathbf{W} + \alpha \mathbf{1} \mathbf{u}^\top)
$$

每一步都可能按照指定的概率到达每一个节点

### Random Walk with Restart (RWR)

$$
\mathbf{r}^\top=(1-\alpha)\mathbf{r}^\top\mathbf{W} + \alpha \mathbf{u}^\top
$$

每一步都有一定概率直接回到出发的节点，出发的概率是$\mathbf{u}$指定的。在这种情况下，显然RWR和PPR是等价的：

$$
\mathbf{r}^\top((1-\alpha)\mathbf{W} + \alpha \mathbf{1} \mathbf{u}^\top)=(1-\alpha)\mathbf{r}^\top\mathbf{W}+\alpha\mathbf{r}^\top\mathbf{1}\mathbf{u}^\top=(1-\alpha)\mathbf{r}^\top\mathbf{W}+\alpha\mathbf{u}^\top
$$

被用来计算两个节点的相关性，比如计算节点$j$相对于$i$的相关性，那么

$$
\mathbf{r}^{(i)\top}=(1-\alpha)\mathbf{r}^{(i)\top}\mathbf{W} + \alpha \mathbf{e}^{(i)^\top}
$$

$\mathbf{r}^{(i)}_j$就是要的相关性

一般，$\mathbf{r}^{(i)}_j\neq \mathbf{r}^{(j)}_i$，因为$\mathbf{r}^{(i)}_j=\mathbf{e}^{(j)\top}\alpha(\mathbf{I}- (1-\alpha)\mathbf{W})^\top\mathbf{e}^{(i)}$

## HAR

## GENI

## MultiImport
[MultiImport Inferring Node Importance in a Knowled](../Papers/MultiImport%20Inferring%20Node%20Importance%20in%20a%20Knowled.md)

## IPM2016