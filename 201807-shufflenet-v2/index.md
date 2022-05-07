# ShuffleNet V2, Practical Guidelines for Efficient CNN Architecture Design

摘要：ShuffleNet v2中以内存访问代价（Memory Access Cost，MAC）和GPU并行性的方向分析网络效率。
<!--more-->

# ShuffleNet V2: Practical Guidelines for Efficient CNN Architecture Design[^01]

## 文献信息

| 信息               | 内容                                                                                                           |
| ------------------ | ----------------------------------------------------------------------------------------------------------------- |
| 日期               | 2018.07                                                                                                       |
| 作者               | Ningning Ma et al.                                                                                                        |
| 机构               | Megvii Inc (Face++)                                                                                                          |
| 来源               | arXiv                                                                                                           |
| 链接               | [ShuffleNet V2: Practical Guidelines for Efficient CNN Architecture Design](https://arxiv.org/abs/1807.11164v1)                                                                                                        |
| 代码               | [Code]()                                                                                                          |

## 个人理解

>
><strong style="color:red;">问题:</strong> 轻量化网络；
>
><strong style="color:red;">方法:</strong> 高效模型的设计准确；
>
><strong style="color:red;">结论:</strong> 在速度和精度方面达到最先进；
>
><strong style="color:red;">理解:</strong> 
>
>1. 1x1卷积：平衡输入和输出的通道大小->通道相同；
>2. 分组卷积：注意分组数-> 1x1卷积；
>3. 避免网络的碎片化：多路结构->通道分离；
>4. 减少元素级运算：激活函数和add->concat；
>
><strong style="color:red;">优化：</strong>无。
---

## 背景知识

目前衡量模型复杂度的一个通用指标是FLOPs，具体指的是multiply-add数量，但是这却是一个间接指标，因为它不完全等同于速度。如图1中的（c）和（d），可以看到相同FLOPs的两个模型，其速度却存在差异。这种不一致主要归结为两个原因，首先影响速度的不仅仅是FLOPs，如内存使用量（memory access cost, MAC），这不能忽略，对于GPUs来说可能会是瓶颈。另外模型的并行程度也影响速度，并行度高的模型速度相对更快。另外一个原因，模型在不同平台上的运行速度是有差异的，如GPU和ARM，而且采用不同的库也会有影响。

## 原理方法

### 1、高效模型的设计准则

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220507152526713.png width=75% />
</div>


作者在特定的平台下研究ShuffleNetv1和MobileNetv2的运行时间，并结合理论与实验得到了4条实用的指导原则：

G1）：**同等通道大小最小化内存访问量**。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220507202702420.png width=75% />
</div>

对于轻量级CNN网络，常采用深度可分割卷积（depthwise separable convolutions）和点卷积（pointwise convolution），其中1x1卷积复杂度最大。

输入Feature Map的尺寸是 $w \times h \times c_1$，输出Feature Map的尺寸为$w \times h \times c_2$。1x1卷积操作的FLOPs为 $F=h \times w \times c_1 \times c_2 \times 1 \times 1$。在计算这个卷积的过程中，输入Feature Map占用的内存大小是$whc_1$ ，输出Feature Map占用的内存是 $whc_2$ ，卷积核占用的内存是 $c_1c_2$，则：
$$
\begin{aligned}
MAC &= hw(c_1+c_2) + c_1c_2 \\
    &= \sqrt{(hw(c_1+c_2)) ^2} + \frac{B}{hw} \\
    &= \sqrt{(hw)^2(c_1+c_2) ^2} + \frac{B}{hw}	\\
    &\ge  \sqrt{(hw)^2 \times 4c_1c_2} + \frac{B}{hw} \\
    &= s\sqrt{hw \times (hwc_1c_2)} + \frac{B}{hw} \\
    &= 2 \sqrt{hwB} + \frac{B}{hw}
\end{aligned}
$$
其中，$B = hwc_1 c_2$。根据均值不等式，固定$B$时，MAC存在下限且$c_1=c_2$时上式取等号。也就是说当FLOPs确定的时候， $c_1=c_2$时模型的运行效率最高，因为此时的MAC最小。

G2）：**过量使用组卷积会增加MAC**。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220507202846334.png width=75% />
</div>

在分组卷积中，FLOPs为 $F = hw\frac{c_1}{g} \frac{c_2}{g} g = \frac{hwc_1c_2}{g}$ ，其MAC的计算方式为：
$$
\begin{aligned}
MAC &= hw(c_1 +c_2) + \frac{c_1}{g} \frac{c_2}{g} g \\
    &=  hwc_1 + \frac{Bg}{c_1} + \frac{B}{hw}
\end{aligned}
$$

其中，$B=hwc_1c_2/g$，则MAC与$g$成正比；

G3）：**网络碎片化会降低并行度**，多路结构

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220507205309789.png width=75% />
</div>

一些网络如Inception，以及Auto ML自动产生的网络NASNET-A，它们倾向于采用“多路”结构，即存在一个lock中很多不同的小卷积或者pooling，这很容易造成网络碎片化，减低模型的并行度，相应速度会慢，这也可以通过实验得到证明。

G4）：**不能忽略元素级操作**

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220507205251913.png width=75% />
</div>

对于元素级（element-wise operators）比如ReLU和Add，虽然它们的FLOPs较小，但是却需要较大的MAC。这里实验发现如果将ResNet中残差单元中的ReLU和shortcut移除的话，速度有20%的提升。

### 2、ShuffleNet v2 单元结构

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220507152441689.png width=75% />
</div>

（a）普通ShuffleNet 单元；

（b）下采样ShuffleNet 单元；

（c）改进ShuffleNet 单元；

- 通道分离（Channel Split），即将$c$个输入Feature分成 $c-c'$和 $c'$ 两组，一般情况下$c'= \frac{c}{2}$ ，这种设计是为了尽量控制分支数，为了满足G3。
- 左侧是一个直接映射；
- 右侧的卷积中，1x1卷积并没有使用分组卷积，为了满足G2。
- 右侧的卷积中，一个输入通道数和输出通道数均相同的深度可分离卷积，为了满足G1。
- 合并的时候均是使用拼接操作，为了满足G4。
- 在堆叠ShuffleNet v2的时候，通道拼接，通道洗牌和通道分割可以合并成1个element-wise操作，也是为了满足G4。

（d）改进下采样ShuffleNet 单元：当需要降采样的时候，通过不进行通道分割的方式达到通道数量的加倍；

### 3、特征重用

ShuffleNet v2能够得到非常高的精度是因为它和DenseNet有着思想上非常一致的结构：强壮的特征重用（Feature Reuse）。在DenseNet中，作者大量使用的拼接操作直接将上一层的Feature Map原汁原味的传到下一个乃至下几个模块。而通道分离再拼接，左侧的直接映射和DenseNet的特征重用是非常相似的。

不同于DenseNet的整个Feature Map的直接映射，ShuffleNet v2只映射了一半。恰恰是这一点不同，是ShuffleNet v2有了和DenseNet的升级版CondenseNet相同的思想。在CondenseNet中，作者通过可视化DenseNet的特征重用和Feature Map的距离关系发现距离越近的Feature Map之间的特征重用越重要。ShuffleNet v2中第 i个和第 i+j个Feature Map的重用特征的数量是$\frac{c}{2^j}$。也就是距离越远，重用的特征越少。

### 4、ShuffleNet v2 网络结构

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220507153104836.png width=75% />
</div>

## 实验结果
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220507153127980.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220507153144423.png width=75% />
</div>


## 参考文献

[^01]: [大师兄-ShuffNet v1 和 ShuffleNet v2-知乎](https://zhuanlan.zhihu.com/p/51566209)
[^02]: [小小将-ShuffleNetV2：轻量级CNN网络中的桂冠-知乎](https://zhuanlan.zhihu.com/p/48261931)


