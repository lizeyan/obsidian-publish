---
share: true
tags:
  - Paper Notes
  - WWW
  - "2017"
---



## Introduction

云服务（PaaS）因为其对资源的抽象带来的便利，被越来越多地使用，尤其是web service。但是因其对资源的抽象，在发生故障时诊断故障变得很困难。

应用开发者需要对应用进行异常检测，根因定位等，这需要云服务商提供数据采集和分析能力，但是大多数云服务商没有提供。

有很多在应用层面收集数据和分析的工具包，但是这增加了成本，并且需要让云服务商信任这些工具是正确使用的。另一方面，由于某些资源的抽象（比如数据库），这样的程序包也不能完全做到故障定位。

现代的PaaS平台多包含很多层次，很多组件，除了数据收集之外，还需要足够的分析能力来定位哪个组件才是根因。

这篇文章提出的Roots，是一个application platform monitor （APM），作为PaaS的一部分服务来运行。可以检测异常， 分析根因和瓶颈。

这篇文章在开源的PaaS平台AppScale进行了评测。

这是一个system领域的工作，主要的工作是构建了一个PaaS中的系统

## Methodology

主要的有几部分

1. 数据收集
2. 响应的异常检测
3. workload变化检测
4. bottleneck分析

数据收集是指收集每个调用在每一部分的响应时间。为了实现这一点，Roots需要修改frontend server的实现，在入口的HTTP header里面加上一个request id，这种方法只适用使用HTTP/HTTPS做API调用的系统。具体做法是在PaaS的每个组件上，用异步的方法记录下收到每个request id的时间。然后会有pod定时收集不同组件上的数据。

响应的异常检测。基于SLO。实际上就是阈值而已。

workload变化。使用 change point detection 的方法检测workload的变化。用处是，如果异常（响应时间超过阈值）的同时用workload的变化，那么就把异常归因与workload的变化，不触发bottleneck分析。

bottleneck分析。提出四种方法对组件进行排序，最后通过majority vote确定最后的bottleneck。因为是四种方法，所以给第一种relative improtance的方法更高的权重，来保证总是能给出唯一的结果

1. relative importance。分析每个组件的响应时间对总体响应时间的variance的贡献
2. change in relative importance
3. 0.99 quantile
4. 最大值和0.99 quantile的比例

## Evaluation

在AppScale上进行的了测试，运行了几个不同的应用。

一个是guestbook，会使用两个PaaS服务；另一个是stock trader，会使用8个PaaS服务。

这个系统会有一些内在的异常，另外会每一个小时注入一个响应时间增加的故障

###### 异常检测

实验只说明了在有注入的异常的情况下，SLO也能检测出来其他异常；另外说明了SLO阈值越大检测出来的就越少（显然是这样）。因滑动窗口的原因，会有误报的

还评测了检测速度

###### workload change analyze

注入了5个workload burst，都能检测出来，并且没有误报。不过平时的workload太平稳，bursting的workload太剧烈，评测的数据太简单了

![Untitled](../../attachments/Untitled.png)

#### Bottleneck

（前面的异常检测的实验）在guestbook应用中的database微服务注入了故障，10个异常中只有一个没有被定位为database的问题。但是考虑到异常很明显，而且只有两个微服务，这个结果不是很能接受。

在更复杂的应用上也做了类似的实验，结果也是至多有一个异常没有正确定位到故障微服务

## Thinking and Conclusion

文章最主要的贡献就是提出了一个PaaS平台上做异常定位和瓶颈检测的系统。但是作为PaaS开发者的话，PaaS平台也就是一个普通的多组件应用而已，这篇文章的看起来也没有针对PaaS平台做什么特别的工作。

文中使用了一些方法来做异常检测，瓶颈检测等工作。异常检测用的是阈值，workload change 检测直接用了别的工作的方法，瓶颈分析用了很大篇幅介绍了四个方法，但是后面的评测看不出来这四个方法到底分别有多有用。

在评测里只是说明了这个系统能work而已。但是对于检测、定位的效果并没有和baseline的对比。这个系统并不是第一个能在他们的场景下work的异常检测、瓶颈定位工作。

这个文章难道就是报告一下他们实现了一个系统么？可是也不是在生产环境应用了产生效果的，只是在很小的testbed上做了实验。