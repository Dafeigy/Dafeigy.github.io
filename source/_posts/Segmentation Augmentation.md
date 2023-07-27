---
title: 语义分割任务的数据增强方式
tags:
- 计算机视觉
- 数据增强
- TODO
categories: 
- 计算机视觉
---

记录了一些视觉任务中的图像处理方式。

<!-- more -->

# 语义分割中数据增强方式（待补充）

## EfficientNet-L2+NAS-FPN

VOC PASCAL 达到90.5%mIoU，次SOTA，2020年

* standard Flip ＆ Crop：包含了水平翻转和尺度抖动。水平翻转很常规，尺度抖动（Scale Jittering）即将不规则大小的img按照短边等比例缩放到某一个区间（一般为`crop_size`的$[0.8,1.2]$），并对等比例缩放后的结果进行固定尺寸大小的`crop_size`进行裁剪。

* 更大尺度的村都抖动：这篇文章里将尺度抖动放大到了$[0.5,2]$。

* Auto Augment：[[1805.09501\] AutoAugment: Learning Augmentation Policies from Data (arxiv.org)](https://arxiv.org/abs/1805.09501)：是一种强化学习探索得到的针对数据集适合的数据增强方式，尽管达到当时的SOTA，但是参考价值不大。具体来说是构建一系列的基本图像数据增强方式：

  ![image-20221014135442552](C:\Users\10706\AppData\Roaming\Typora\typora-user-images\image-20221014135442552.png)

  随后通过PPO的迭代找到最优的策略操作，一共有两个operation，operation将从上面的增强方式中进行选取。

* RandAugment：[[1909.13719\] RandAugment: Practical automated data augmentation with a reduced search space (arxiv.org)](https://arxiv.org/abs/1909.13719)：

  首先讲讲自动数据增强方法中的”随机性“。以AutoAugment为例，通过搜索得出的最佳增强策略中，有多个（假设有K个）子策略，然后在训练时，对于某一批次的数据，从这K个中随机选择一个进行应用（这是随机性的第一层体现）。每个子策略由两种增强操作构成，每个操作有概率和幅度两个参数，也就是说，即使选中了这个子策略，这一批次的数据有没有被增强，或者经过这两个操作中哪个的增强，都由该子策略中的这两个概率参数控制（这是随机性的第二层体现）。即使应用了某一操作，比如旋转，虽然幅度参数指定了增强的幅度（比如说旋转45度），但具体是逆时针还是顺时针同样也是随机的（这是随机性的第三层体现）。

  对此，作者舍弃掉这一系列的随机性（按我的理解是，这些反复的随机性，其实是削弱了AutoAugment中概率和幅度参数的作用。因此作者认为舍弃掉概率参数，或者说，将这些参数设为统一的值，从而免去搜索这些值的时间，换来效率的提高是值得的），直接将每种操作的应用概率设置为一样。因此，RandAugment方法为：

  1. 设定一个操作集，例如作者的操作集由14种操作构成：Identity、AutoContrast、Equalize、Rotate、Solarize、Color、Posterize、Contrast、Brightness、Sharpness、ShearX、ShearY、TranslateX、TranslateY。

  2. RandAugment只有两个参数：N和M。其中N为在每次增强时使用N次操作（使用的这N个操作，都是从操作集中等概率抽取的，例如操作集中有14种操作，则每种操作被选中的概率为1/14，每张图像的N次增强中，选到的操作可能是一样的），M为正整数，表示所有操作在应用时，幅度都为M。

  3. 使用网格搜索，或者更为高端的方法（如反向传播等）在完整数据集、完整网络上实验，找到最适合的N和M。这样一来，假如说N的搜索空间为1和2，M为1至10，则搜索空间仅为10^2，远小于之前的自动增强方法。

  通过更改N、M的值，便能控制训练时的正则化强度。N、M越大，正则化强度则越高。