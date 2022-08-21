# ShuffleNet, An Extremely Efficient Convolutional Neural Network for Mobile Devices

摘要：ShuffleNet 解决分组卷积的计算量和特征通信问题。
<!--more-->

# ShuffleNet: An Extremely Efficient Convolutional Neural Network for Mobile Devices[^01]

## 文献信息

| 信息               | 内容                                                                                                           |
| ------------------ | ----------------------------------------------------------------------------------------------------------------- |
| 日期               | 2017.01                                                                                                     |
| 作者               | Xiangyu Zhang et al.                                                                                                        |
| 机构               | Megvii Inc (Face++)                                                                                                          |
| 来源               | arXiv                                                                                                           |
| 链接               | [ShuffleNet: An Extremely Efficient Convolutional Neural Network for Mobile Devices](http://link.zhihu.com/?target=https%3A//arxiv.org/abs/1707.01083)                                                                                                        |
| 代码               | [Code]()                                                                                                          |

## 个人理解
><strong style="color:red;">问题:</strong> 文章为了分组卷积的计算量和特征通信问题；
>
><strong style="color:red;">方法:</strong> 文章提出了1x1分组卷积 + 通道洗牌；
>
><strong style="color:red;">结论:</strong> 在计算预算为40 MFLOPs的情况下，在ImageNet分类任务上比最近的MobileNet更低的top-1错误率（绝对7.8%）。在基于ARM的移动设备上，ShuffleNet通过AlexNet实现了约13倍的实际加速，同时保持了相当的精度。；
>
><strong style="color:red;">理解:</strong> DW卷积虽然能够有效降低计算量，但是缺少通道间的信息交互与整合，影响网络的特征提取能力，MobileNet使用PW卷积来解决这个问题，但PW卷积的计算量还是比较大（相对dw卷积）。作者提出使用逐点分组卷积（pointwise group convolution）和通道混洗(channel shuffle)来解决1x1卷积复杂度高的问题。
>
><strong style="color:red;">优化：</strong>无。
---

## 背景知识

1、分组卷积的矛盾——计算量

使用group convolution的网络有很多，如MobileNet，Xception，ResNeXt等。其中Xception和MobileNet采用了depthwise convolution，这是一种比较特殊的group convolution，此时分组数恰好等于通道数，意味着每个组只有一个特征图。但这些网络存在一个很大的弊端是采用了密集的1x1 pointwise convolution。

2、分组卷积的矛盾——特征通信

group convolution层另一个问题是不同组之间的特征图需要通信，否则就好像分了几个互不相干的路，大家各走各的，会降低网络的特征提取能力，这也可以解释为什么Xception，MobileNet等网络采用密集的1x1 pointwise convolution，因为要保证group convolution之后不同组的特征图之间的信息交流。

## 原理方法

### 1、Channel Shuffle 通道洗牌

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220507111612434.png width=75% />
</div>
原理：深度可分离卷积+1x1点卷积 => 分组卷积+channel shuffle，前者降低参数，后者达到特征通信，实现信息流通。

参数：
$$
9 \times h \times w \times c_1 + \frac{h \times w \times c_1 \times c_2}{1} =>

9 \times h \times w \times c_1 + \frac{h \times w \times c_1 \times c_2}{g} 
$$
一般情况下c2是远大于9的，也就是说深度可分离卷积的性能瓶颈主要在Pointwise卷积上；为了解决这个问题，ShuffleNet v1中提出了仅在分组内进行Pointwise卷积，对于一个分成了g个组的分组卷积。但是可以看出组内Pointwise卷积可以非常有效的缓解性能瓶颈问题。然而这个策略的一个非常严重的问题是卷积直接的信息沟通不畅，网络趋近于一个由多个结构类似的网络构成的模型集成，精度大打折扣。



在程序上实现channel shuffle：假定将输入层分为 g 组，总通道数为 $g\times n$ ，将通道那个维度拆分为 (g,n) 两个维度，然后将这两个维度转置变成 (n,g) ，最后重新reshape成一个维度$g\times n$。

```python
'''Channel shuffle: [N,C,H,W] -> [N,g,C/g,H,W] -> [N,C/g,g,H,w] -> [N,C,H,W]'''
N,C,H,W = x.size()
# 维度变换之后必须要使用.contiguous()使得张量在内存连续之后才能调用view函数
return x.view(N, g, int(C/g), H, W).permute(0, 2, 1, 3, 4).contiguous().view(N, C, H, W)
```

### 2、ShuffleNetv1 网络结构

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220507114526649.png width=75% />
</div>

(a) MobileNet: 带有残差结构的深度可分离卷积;

(b) ShuffleNetv1 残差块：1x1分组卷积 + channel shuffle + 3x3深度可分离卷积 + 1x1分组卷积，残差连接；
- 1x1 卷积替换为1x1的分组卷积，分组g一般不会很大，论文中的几个值分别是1，2，3，4，8。当$g=1$时，ShuffleNet v1退化为Xception。g的值确保能够被通道数整除，保证reshape操作的有效执行；
- 添加Channel Shuffle操作；
- 去掉3x3深度可分离卷积之后的ReLU激活，目的是为了减少ReLU激活造成的信息损耗；

(c) ShuffleNetv1 降采样块：1x1分组卷积 + channel shuffle + 3x3深度可分离卷积s=2 + 1x1分组卷积，残差连接s=2平均池化；

- 降采样的情况，左侧shortcut部分使用的是步长为2的3x3平均池化，右侧使用的是步长为2的 3x3 的Depthwise卷积；
- 降采样，为了保证参数数量不骤减，往往需要加倍通道数量,使用的是拼接（Concat）操作用于加倍通道数；

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220507114544831.png width=75% />
</div>

单元计算量FLOPs：Feature Map的尺寸为cxhxw，bottleneck的通道数为m，$F_{Resnet} > F_{ResNext} > F_{ShuffleNet}$

- ResNet

$$
1 \times 1 \times hwcm + 3 \times 3 \times hwmn + 1 \times 1 \times  hwcm =  hw(2cm + 9m^2)
$$

- ResNeXt

$$
1 \times 1 \times hwcm + 3 \times 3 \times hw \frac{m}{g} \times \frac{m}{g} \times g + 1 \times 1 \times hwcm =  hw(2cm + \frac{9m^2}{g})
$$

- ShuffleNet

$$
hw\frac{c}{g}\frac{m}{g} \times g  + 3 \times 3 \times hwm + hw \frac{c}{g} \frac{m}{g} \times g = hw(\frac{2cm}{g} + 9m)
$$

## 训练测试

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220507114556679.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220507114613382.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220507114631117.png width=75% />
</div>
## 参考文献

[^01]: [大师兄-ShuffNet v1 和 ShuffleNet v2-知乎](https://zhuanlan.zhihu.com/p/51566209)




