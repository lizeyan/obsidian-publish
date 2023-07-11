---
share: true
tags:
  - "Paper Notes"
  - "NSDI"
  - "2022"
---

集群上任务（Job）的调度是一个重要的问题，任务调度的核心是预测每一个任务需要多少时间才能完成。
现有的工作大多数基于已有的相同任务的执行时间学习。这种做法依赖的假设是任务重复发生，且相同任务重复执行时，有着类型的特性。但是这两个假设都是不对的，很多工作证明了，重复发生的任务没有那么多；同一个任务的代码、集群的硬件情况都是快速发生变化的。
因此，这篇文章提出，可以通过一个任务内部的部分task的执行时间，预测剩下的task的执行时间。本文称之为learing in space（相比之下，之前的模式称之为learning in time）
通过一些分析可以发现，如果job和job之间的variation大于同一个job内部task和task的variation，那么learning in space会比learning in time更准确。本文通过实证分析证明了这两点。
但是，learning in space需要在每个任务执行之前，采样其中的一部分task并统计其执行时间，这带来了较大的延迟。因此，我们还关注，learning in space带来的准确率提升（进而带来的更好的scheduling），是否能够抵消learning in space带来的额外延迟。本文通过一些case分析证明了这一点。
最后，本文提出了一个针对DAG job的hybrid方法。因为DAG job中任务存在依赖关系，因此必须每个stage都进行采样才能进行learning in space，这样延迟就太大了。为此，本文只在第一个stage进行learning in space，然后把学到的learning in space和learning in time的误差，等比例应用到后面的space，从而取得delay和accuracy的权衡。