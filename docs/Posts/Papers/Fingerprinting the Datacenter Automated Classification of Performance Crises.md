---
share: true
tags:
  - Paper Notes
  - EuroSys
  - "2010"
---


数据中心中发生的故障需要花费很长时间去排查和止损、修复(可能多达一小时或更多). 由于种种原因, 这些故障可能是重复发生的 (比如因为修复方案还未实施、修复方案有误、故障是由于高负载等外部因素引起等等). 所以,如果在运维人员开始去排查故障之前, 我们能够提示运维人员这个故障可能和之前的某次故障有关, 那么就能让运维人员直接参考之前的止损方案, 从而减少TTM (time to mitigate). 因此这篇文章提出了一个叫fingerprint的相似故障推荐算法.

Fingerprint面对的场景是一个大型的单体应用, 部署在很多不同的服务器上, 每个服务器跑的代码是相同的. 整个应用会有一些KPI, 基于这些KPI定义了一些SLA. 我们故障止损的目标就是恢复SLA. 描述故障用的数据来自每个服务器上采集到的各种指标, 包括OS级, 应用级, 硬件级, 中间件级等等各种指标.

由于不同的服务器没有什么本质不同, 因此fingerprint对故障进行描述是基于指标的. 对于一个指标, fingerprint把它在不同服务器上取值的quantile作为这个指标的代表值, metric quantile.

> We observe that the main factor that differentiates between different types of crises are the different metric quantiles that take extreme values during the crisis. In other words, compared to values of a metric quantile during normal periods with no crises, the values observed during a crisis are either too high or too low.
> 

根据这段总结, fingerprint 通过 metric quantile 是否太大或者太小来描述一个故障. 具体来说, fingerprint会比较metric quantile和历史值的百分位数, 如果太大那么对应的位就是1, 太小就是-1, 否则就是0. 

然后fingerprint会选择有用的metric. 对于每个故障, fingerprint会将每个服务器的指标作为feature, KPI作为target, 训练一个L1正则的LR模型, 然后选择最有用的指标. 然后对于所有故障, 选择被认为有用的频率最高的指标.

最后就只需要对useful metrics的状态(-1, -, 1)向量比较欧氏距离就可以了.

*fingerprint整体用到的方案都非常简单,计算复杂度低,没有什么参数. 但是它的局限在于只能用在不同的服务器完全互相可替换的情况, 如果不同的服务器跑的是不同的代码,或者配置不同, 那么就很难用这种方案去做fingerprinting.*