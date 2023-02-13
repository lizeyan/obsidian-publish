---
share: true
tags:
  - Paper Notes
  - NOMS
  - "2020"
---


1. 通过调用关系构建调用图。其中还包括了服务部署于host上的边。
2. 对边进行异常检测，留下有异常的节点以及和它们相连的节点组成的子图
3. 计算边的权重和节点的分数
4. PPR

![Untitled](../../attachments/Untitled.png)

![Untitled 1](../../attachments/Untitled%201.png)

![Untitled 2](../../attachments/Untitled%202.png)

相比MonitorRank的优势是能够避免RC和frontend离得远导致的关联性不强的情况

> This is because MonitorRank calculates the similarity based on the correlation between front-end services and backend services. The anomaly of microservice payment decreases the correlation during propagation and thus the root cause localization fails.
> 

相比MicroScope的优势是能检测出来那种非末端叶节点的根因

> This is because Microscope traverses the graph based on the detected anomalies and put the anomalous child nodes into a list of potential causes, which makes it fail to identify the dominating nodes when alarms are reported from child nodes.
>