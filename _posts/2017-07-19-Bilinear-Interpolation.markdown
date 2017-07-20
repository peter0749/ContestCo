---
layout: post
title: 圖像縮放 -- Bilinear Interpolation, Nearest Neighbor
date: 2017-07-20 23:00
comments: true
external-url:
tags: image_processing
---

再來練習個簡單的圖像縮放，

使用的方法有 Bilinear Interpolation 和 Nearest Neighbor

## 結果

Before: 

![img1](https://github.com/peter0749/Image_processing_practice/raw/master/ZJb424_scaling/miku.png)

After:

![img2](https://github.com/peter0749/Image_processing_practice/raw/master/ZJb424_scaling/miku_big.png)

放大

![img3](https://github.com/peter0749/Image_processing_practice/raw/master/ZJb424_scaling/miku_small.png)

縮小

![img4](https://github.com/peter0749/Image_processing_practice/raw/master/ZJb424_scaling/miku_change_ratio.png)

不照比例縮放

上面的例子都是使用 Bilinear Interpolation 做成的，
Nearest Neighbor 方法跑出來的結果就不放上來了。

