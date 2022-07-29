# MobileNetV2, Inverted Residuals and Linear Bottlenecks

摘要：MobileNet v2网络是由google团队在2018年提出的，相比MobileNet V1网络，准确率更高，模型更小，创新点是 Inverted Residuals（倒残差结构）和Linear Bottlenecks（结构的最后一层采用线性层）。
<!--more-->

## 文献信息
| 信息 | 内容                                                         |
| ---- | ------------------------------------------------------------ |
| 日期 | 2018.01                                                      |
| 作者 | Mark Sandler et al.                                          |
| 机构 | Google Inc                                                   |
| 来源 | arXiv.                                                       |
| 链接 | [MobileNetV2: Inverted Residuals and Linear Bottlenecks](https://arxiv.org/abs/1801.04381) |
| 代码 | [Code]()                                                     |

## 个人理解
><strong style="color:red;">问题:</strong> ReLU激活问题和特征信息缺失；
>
><strong style="color:red;">方法:</strong> 逆残差块 + 线性瓶颈块 + ReLU6；
>
><strong style="color:red;">结论:</strong> 在ImageNe分类、COCO目标检测和VOC图像分割方面衡量了性能，并评估了精度、乘加（MAdd）测量的操作数、实际延迟和参数数之间的权衡。；
>
><strong style="color:red;">理解:</strong> 
>
>1. 从先降维到先升维，提高特征提取能力，升维通过扩充因子控制，为6。
>2. ReLU激活实验。
>3. ReLU6。
>4. MobileNet v2类似纺锤形块，将一个卷积操作继续分解为三个乘法的和，使得每个Block之间可以以较小的通道数[24 32 64-96 160-320]进行传递，二来在Block中升维[144 192 384-576 960-1920]可以学习到更多特征。将CONV1x1层的参数数量和计算量直接减小若干倍（论文中6倍），轻微增加DWCONV3x3的通道数以保证的网络容量。
>
><strong style="color:red;">优化：</strong>无。
---

## 背景知识

1. 针对高维的输入，ReLU能够保留输入的主要信息，而会破坏低维的输入信息。

维度低的feature，分布到ReLU的激活带上的概率小，因此经过后信息丢失严重，甚至可能完全丢失。而维度高的feature，分布到ReLU的激活带上的概率大，虽然可能也会有信息的部分丢失，但是无伤大雅，大部分的信息仍然得以保留。在这条准则下，

最好的方式就是，在ReLU之前进行升维。而降维后就要不使用ReLU。

2. 但是又有个问题，如果主要信息在ReLU变换后保持完整，则、那么ReLU实质上退化为了线形变换。

因此，利用残差结构，让线形的的部分通过残差的支路传播（如果输出为线形，则ReLU退化为线形变化，则不存在激活这一说法了）。非线形的部分通过Inverted Residual Block来处理（如果输出为非线性，则经过ReLU会损失）。

## 原理方法

### 1、Inverted Residuals

ResNet残差结构是先用1x1的卷积降维，再升维的操作，其目的是降低参数量。

MobileNetV2中，先升维，在降维的操作。先升维度的原因是浅层通道数很少，DW只能在低维空间提特征，因此升维可以增强DW提取特征能力。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213912.png width=75% />
</div>

### 2、Linear Bottlenecks

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213913.png width=75% />
</div>


图(a)：普通模型；图(b)：MobileNetv1中深度卷积和逐点卷积；图（c）和图(d)是MobileNetv2的结构(d是c的下一个连接状态)，同样是将标准卷积拆分为深度卷积和逐点卷积，在逐点卷积后使用了接1 × 1卷积，该卷积使用线性变换，总称为一层低维linear bottleneck，其作用是将输入映射回低维空间。

**针对倒残差结构中，最后一层的卷积层，采用了线性的激活函数，而不是ReLU激活函数。**

一个解释是，ReLU激活函数对于低维的信息可能会造成比较大的瞬损失，而对于高维的特征信息造成的损失很小。而且由于倒残差结构是两头小中间大，所以输出的是一个低维的特征信息。所以使用一个线性的激活函数避免特征损失。

”线性流度“概念，实验如下：嵌入高维空间的低维流形的ReLU变换，使用随机矩阵T和ReLU将初始螺旋嵌入n维空间，然后使用$T^{-1}$将其投影回2D空间.在上述示例中，n=2，3导致信息丢失，流形的某些点彼此塌陷，而对于n=15到30，变换高度非凸。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213914.png width=75% />
</div>

### 3、Expansion factor(扩展系数)

到底需要升维多少通道呢，引进了扩展系数$t$，linear bottleneck到深度卷积之间的的维度比称为**Expansion factor(扩展系数)，该系数t(默认为6)控制了整个block的通道数。**

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213915.png width=75% />
</div>


### 4、ReLU6 激活函数

$$
ReLU6 = min(max(x, 0), 6)
$$

ReLU6 就是普通的 ReLU 但是限制最大输出为6，这是为了在移动端设备 float16/int8 的低精度的时候也能有很好的数值分辨率。如果对 ReLU 的激活范围不加限制，输出范围为 0 到正无穷，如果激活值非常大，分布在一个很大的范围内，则低精度的float16/int8 无法很好地精确描述如此大范围的数值，带来精度损失。

### 5、MobileNetV2 整体结构

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213916.png width=75% />
</div>

两种逆残差结构：

（1）当stride=1且 输入特征矩阵与输出特征矩阵shape 相同时才有shortcut连接；

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213910.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213911.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213911.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213912.png width=75% />
</div>

其中，shape的变化：其中的t是扩充因子。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213916.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213917.png width=75% />
</div>


## 实验结果

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213917.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213918.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213919.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213919.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213920.png width=75% />
</div>


## 参考文献
[^01]: [黄鑫元-详解MobileNetV2-知乎](https://zhuanlan.zhihu.com/p/98874284)


