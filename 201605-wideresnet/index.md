# Wide Residual Networks

摘要：WideResNet（WRN），2016年Sergey Zagoruyko发表，从增加网络宽度角度改善ResNet，性能和训练速度都提升。
<!--more-->

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220511153951641.png width=75% />
</div>


# Wide Residual Networks

## 文献信息

| 信息 | 内容                                                         |
| ---- | ------------------------------------------------------------ |
| 日期 | 2016.05                                                      |
| 作者 | Sergey Zagoruyko et al.                                      |
| 机构 | Université Paris-Est, École des Ponts ParisTech              |
| 来源 | 2016 CVPR                                                    |
| 链接 | [Wide Residual Networks](https://arxiv.org/abs/1605.07146)   |
| 代码 | [szagoruyko/wide-residual-networks](https://github.com/szagoruyko/wide-residual-networks) |

## 个人理解
><strong style="color:red;">问题:</strong> 神经网络是否越瘦越深则结果越好？ 还是说只要能保证参数的数量，训练一个更胖更浅的网络 是否可行？
>
><strong style="color:red;">方法:</strong> WideResNet + Dropout。
>
><strong style="color:red;">结论:</strong> 即使是一个简单的16层深宽残差网络，其精度和效率也优于所有以前的深残差网络，包括千层深宽残差网络，在CIFAR、SVHN、COCO上取得了最新的成果，并在ImageNet上取得了显著的改进。
>
><strong style="color:red;">理解:</strong> 网络不仅仅越来越深，网络更宽也是一条道路，理论很简单，但往往简单就越有效。
>
><strong style="color:red;">优化：</strong>Dropout优化可能还存在疑惑。
---

## 背景知识
第一个问题，随着研究的深入，神经网络越来越更深且更瘦，那么神经网络是否越瘦越深则结果越好？ 还是说只要能保证参数的数量，训练一个更胖更浅的网络 是否可行？

> 为什么网络会越来越深，但是会越来越廋呢？因为残余网络的作者试图让网络尽可能薄，为了增加深度的同时保证参数不大规模增加或者减少，因此作者甚至引入了瓶颈块，使ResNet块更薄。

Dropout为什么逐渐被取代？

> Dropout主要是被BN替代，个人理解是因为BN要计算均值和方差，而当你随机使神经元失活，那么每一次输出的方差就会不断变化，从而影响BN的效果。

## 原理方法

### 1、残差块与宽残差块

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220511154804451.png width=75% />
</div>


（a）基础残差块：堆叠卷积并引进跳跃连接；

（b）瓶颈残差块：1x1卷积降维；

注意：虽然（b）比（a）更节省计算时间与计算开销，但是作者想研究宽度对神经网络结果的影响，于是采取结构(a)进行试验。

（c）宽残差块：增加网络通道数量；

（d）带dropout宽残差块：引进dropout：

### 2、残差块结构优化实验

如何简单的提升残差模块的表达能力，作者提出了3个方法：

1. 加层，更多的层带来更多的参数，在理想状态下，更多的参数能够拟合更复杂的结果；

2. 通过增加卷积层的宽度（所谓宽度也就是每个卷积层的通道数）；

3. 换更大的卷积核；

作者比较了不同的卷积核，3X3的卷积核证明十分有效。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220511161628662.png width=75% />
</div>

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220511161657546.png width=75% />
</div>


因此作者以3x3卷积核为基础，对比了网络深度与宽度实验：作者引进网络宽度因子k和网络深度因子N，通过因子k来控制卷积的通道数量，因子N来控制网络的深度，实验表明网络深度为28，k=10或12时结果达到最优。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220511155159554.png width=75% />
</div>


<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220511162132912.png width=75% />
</div>

### 3、Dropout 结果

由于残差块的扩大，导致参数数量的增加，从而容易导致过拟合。因此，作者对正则化进行了研究，作者认为BN归一化需要大量的数据，但是往往这不太可能。因此引进了dropout，即干扰BN和防止过拟合的影响。

作者以前研究残差网络中恒等部分嵌入dropout，结果产生负面影响。相反，在宽幅残差网络上卷积层之间插入dropout，实验结果表明，这种方法可以获得增益。

结构：预激活模式`bn + relu + conv + dropout + bn + relu + conv  + shotcut`。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220511163200887.png width=75% />
</div>

## 实验结果

不同网络实验结果对比：
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220511163036416.png width=75% />
</div>

训练损失和收敛速度比较：
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220511163117180.png width=75% />
</div>
带Dropout训练损失和收敛速度比较：
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220511163219375.png width=75% />
</div>

## 参考文献

[^01]: [作者-文章名-来源](URL)
