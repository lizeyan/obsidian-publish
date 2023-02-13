---
share: true
tags:
  - Paper Notes
  - ATC
  - "2020"
---


## Introduction

很多故障都是配置错误导致的。但是呢，很多配置错误在manual里面已经写清楚了，推荐什么或者不推荐什么。这些一般都是soft constraint（特定情况下这么配是需要的），所以软件也不会检查。

但manual一般都太长了，所以PracExtractor想自动从manual文本中提取推荐配置，并检查配置中的错误

## Methodology

1. 提取推荐配置
    1. 关键词过滤。从label数据集中提取和推荐相关的句子的关键词
    2. pattern 过滤。把句子转化成语法树，然后从label数据集中提取frequent pattern
    3. 生成配置w

## Evaluation

## Thinking and Conclusion