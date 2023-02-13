---
share: true
tags:
  - Paper Notes
  - SIGMOD
  - "2020"
---


#### 分类

- Lock-step 即点对点比较
- sliding
- elastic 对齐或者拉伸time series
- kernel。需要是半正定的
- embedding

#### 四个midconceptions和对应的结论

###### normalization

z-normalization被普遍默认使用，但是没有文章分析过normalization对distance的影响

本文指出normalization对很多distance都有重要的影响

###### lock-step distance

默认的选项都是欧式距离（ED），因为它实现简单，无参数，高效，有很多研究默认基于ED（lzy：比如有些LSH方法，都是基于ED推导的）

这篇文章发现其他lockstep方法显著地比ED好

![Untitled](../../attachments/Untitled.png)

![Untitled 1](../../attachments/Untitled%201.png)

###### elastic方法是不是总是比sliding方法好

不是，本文发现大部分分elastic方法都没有超越sliding方法。

实际上，如果不通过cross validation方法调参，只有MSSM和TWE比cross correlation更好（DTW都没有做到）

![Untitled 2](../../attachments/Untitled%202.png)

###### DTW是不是最好的elastic方法

不是，MSM和TWE都比DTW好

MSM也基于编辑距离，但是它是一个metric（应该指的是泛函分析的度量概念，DTW和TWE都不是，不满足三角不等式）

![Untitled 3](../../attachments/Untitled%203.png)

TWE结合了LCSS和DTW，也是一个metric

![Untitled 4](../../attachments/Untitled%204.png)