---
share: true
tags:
- PyTorch
---

`th.mean` 和`th.nanmean` 都是计算tensor的均值,不同的时候后者会在计算之前剔除掉所有的`nan` . 但是这导致`th.nanmean` 的复杂度提高

`%timeit th.mean(th.ones(widith, height), dim=-1)`  (illustrating pseudo code)

![Untitled](../../attachments/th nanmean vs th mean_image_1.png)

![Untitled](../../attachments/th nanmean vs th mean_image_2.png)