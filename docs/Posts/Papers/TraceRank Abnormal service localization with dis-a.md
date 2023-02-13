---
share: true
tags:
  - Paper Notes
  - "Journal of Software: Evolution and Process"
  - "2021"
---

这篇文章通过dis-aggregated traces (其实就是单条的traces, 和把trace聚合起来用指标做定位的monitorank等区分, 这一点并不是什么contribution)定位根因微服务. 具体采用SBFL的方法. 为了进一步提升SBFL的效果, 加入了pagerank-based算法去进一步修正SBFL的结果.

TraceRank的方法分成一下几步

首先是对trace进行异常检测. 因为不同类型的trace的latency本来就不同, 所以TraceRank首先用hierarchical聚类方法把traces分成若干类. 对于每一个trace, 通过一个k=2的k-means进行异常检测. 核心思想是, 如果这一类的trace都是正常的, 那么两类的centroid应该相聚很近.

然后是分别使用SBFL和PageRank方法进行排序. 这里和过去的传统方法没有什么不同.

最后是对结果进行修正. TraceRank采用的方法是把两个排序列表中所有整体KPI有异常的服务提取出来并进行concat, SBFL的结果在前.

这篇文章对比中的效果, TraceRank的结果比MonitorRank和T-Rank(同组的一个基于ochai的SBFL定位文章)