# Aggregated Residual Transformations for Deep Neural Networks

摘要：ResNeXt，借鉴VGG、ResNet和Inception的思想组合一下，核心是分组卷积。
<!--more-->

# Aggregated Residual Transformations for Deep Neural Networks

## 文献信息

| 信息 | 内容                                                         |
| ---- | ------------------------------------------------------------ |
| 日期 | 2016.11                                                      |
| 作者 | Saining Xie et al.                                           |
| 机构 | UC San Diego, Facebook AI Research                           |
| 来源 | CVPR 2017                                                    |
| 链接 | [Aggregated Residual Transformations for Deep Neural Networks](https://arxiv.org/abs/1611.05431) |
| 代码 | [acebookresearch](https://github.com/facebookresearch)/[ResNeXt](https://github.com/facebookresearch/ResNeXt) |

## 个人理解
><strong style="color:red;">问题:</strong> 传统的要提高模型的准确率，都是加深或加宽网络，但是随着超参数数量的增加（比如channels数，filter size等等），**网络设计的难度和计算开销**也会增加。
>
><strong style="color:red;">方法:</strong> 文章提出了“同质+多分支”的拓扑结构，并引进了独立于深度和宽度的新维度“基数”，增加容量，控制参数量；
>
><strong style="color:red;">结论:</strong> ILSVRC 2016分类任务第二名；
>
><strong style="color:red;">理解:</strong> 
>
>1. ResNeXt结合ResNet和Inception的思想，但略微不同于Inceptionv4，每一个分支都采用相同的拓扑结构，不需要复杂的结构人工设计。
>
>2. ResNeXt的核心是拓扑结构，拓扑结构的本质是分组卷积，其利用基数（Cardinality）来控制分组的数量。
>
>   注意：分组卷积，“通道独立卷积”，即每个分支产生的Feature Map的通道数为$n(n>1)$ 。
>
>3. 为什么基数提高会提升效果？个人理解，基数越高，容量越大，通道数就更多，每个通道提取的特征不同，通道数越多特征越多，效果也会提升。
>
><strong style="color:red;">优化：</strong>实际场景中，ResNeXt效果低于Incetption，虽然速度增加，但移除了不同感受野的特性，影响精度。
---

## 背景知识

视觉任务从“特征工程”（SIFT/HOG）过渡到“网络工程”（从大规模数据中学习特征），随着超参数（宽度、过滤器、步长等）数量增加，设计架构越来越难；

VGG的堆叠相同形状的构建块思路，被引进到ResNet中，其减少了宽度的超参的设计，仅需要考虑深度的大小。	

**作者认为：规则的简单性可能会降低超参数过度适应特定数据集的风险。**

“分离-转换-合并”思想：Inception模块，可以表征大规模数据特征，同时计算复杂度较低。尽管精度很高，但模型的构建涉及卷积数量和大小设计，模块也是分阶段定制的，因此设计过于复杂。

作者提出了一个简单的体系结构，它采用VGG/Resnet的重复层策略，同时以一种简单、可扩展的方式利用拆分-转换-合并策略，即“相同的拓扑”。

**多分支卷积网络**：Inception 和resnet（一个分支是身份映射）。

**分组卷积**：分组卷积可以追溯到AlexNet。Krizhevsky等人给出的动机是将模型分布在两个GPU上。Caffe、Torch和其他库支持分组卷积，主要是为了与AlexNet兼容。据我们所知，**几乎没有证据表明利用分组卷积来提高准确性**。分组卷积的一种特殊情况是逐通道卷积，其中组数等于通道数，如可分离卷积。

**压缩卷积网络**：分解（空间和/或通道）是一种广泛采用的技术，用于减少深度卷积网络的冗余并加速/压缩它们。

**聚合：**平均一组独立训练的网络是提高准确性的有效解决方案，在识别比赛中被广泛采用。Veit等人将单个ResNet解释为较浅网络的集合，这是ResNet加性行为的结果。作者的方法利用加法来聚合一组转换。**但作者认为将他们的方法视为聚合是不精确的，因为要聚合的成员是联合训练的，而不是独立训练的。**

## 原理方法

ResNeXt:

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220425104313164.png width=75% />
</div>


Inception：split-transform-merge，将输入分配到多路，然后每一路进行转换，最后再把所有支路的结果融合。

ResNeXt：采用每个结构使用相同的拓扑结构，在不增加参数复杂度的前提下提高准确率，同时还减少了超参数的数量。
$$
y = x + \sum_{i=1}^{C} T_i (x)
$$
注意：

1. ResNeXt的分支的拓扑结构是相同的，Inception V4需要人工设计；
2. ResNeXt是先进行1X1卷积然后执行单位加，Inception V4是先拼接再执行1x1卷积，

缺陷：ResNeXt确实比Inception V4的超参数更少，但是他直接废除了Inception的囊括不同感受野的特性仿佛不是很合理，在更多的环境中我们发现Inception V4的效果是优于ResNeXt的。类似结构的ResNeXt的运行速度应该是优于Inception V4的，因为ResNeXt的相同拓扑结构的分支的设计是更符合GPU的硬件设计原则。



代码：`widen_factor=4，[64, 64*4, 128*4, 256*4]， base_width=64`

```python
class ResNeXtBottleneck(nn.Module):
    """
    RexNeXt bottleneck type C (https://github.com/facebookresearch/ResNeXt/blob/master/models/resnext.lua)
    """

    def __init__(self, in_channels, out_channels, stride, cardinality, base_width, widen_factor):
        super(ResNeXtBottleneck, self).__init__()
        width_ratio = out_channels / (widen_factor * 64.)
        D = cardinality * int(base_width * width_ratio)
        self.conv_reduce = nn.Conv2d(in_channels, D, kernel_size=1, stride=1, padding=0, bias=False)
        self.bn_reduce = nn.BatchNorm2d(D)
        self.conv_conv = nn.Conv2d(D, D, kernel_size=3, stride=stride, padding=1, groups=cardinality, bias=False)
        self.bn = nn.BatchNorm2d(D)
        self.conv_expand = nn.Conv2d(D, out_channels, kernel_size=1, stride=1, padding=0, bias=False)
        self.bn_expand = nn.BatchNorm2d(out_channels)

        self.shortcut = nn.Sequential()
        if in_channels != out_channels:
            self.shortcut.add_module('shortcut_conv', nn.Conv2d(in_channels, out_channels, kernel_size=1, stride=stride, padding=0, bias=False))
            self.shortcut.add_module('shortcut_bn', nn.BatchNorm2d(out_channels))

    def forward(self, x):
        bottleneck = self.conv_reduce.forward(x)
        bottleneck = F.relu(self.bn_reduce.forward(bottleneck), inplace=True)
        bottleneck = self.conv_conv.forward(bottleneck)
        bottleneck = F.relu(self.bn.forward(bottleneck), inplace=True)
        bottleneck = self.conv_expand.forward(bottleneck)
        bottleneck = self.bn_expand.forward(bottleneck)
        residual = self.shortcut.forward(x)
        return F.relu(residual + bottleneck, inplace=True)
```

## 训练测试

略。

## 参考文献
[^01]: [大师兄-ResNeXt详解-知乎](https://zhuanlan.zhihu.com/p/51075096)


