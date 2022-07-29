# Xception, Deep Learning with Depthwise Separable Convolutions

摘要：基于Inception v3是假设出发，即解耦通道相关性和空间相关性，进行简化，推导出深度可分离卷积，构建Inception v4网络。
<!--more-->

## 文献信息
| 信息               | 内容                                                                                                           |
| ------------------ | ----------------------------------------------------------------------------------------------------------------- |
| 日期               | 2016.08                                                                                                       |
| 作者               | François Chollet                                                                                                        |
| 机构               | Google, Inc                                                                                                          |
| 来源               | arXiv                                                                                                         |
| 链接               | [Xception: Deep Learning with Depthwise Separable Convolutions](https://arxiv.org/abs/1610.02357)                                                                                                       |
| 代码               | [Code]()                                                                                                          |

## 个人理解
><strong style="color:red;">问题:</strong> 文章为了解决通道之间的相关性与空间相关性问题；
> 
><strong style="color:red;">方法:</strong> 文章提出了深度可分离卷积替代Inception结构；
> 
><strong style="color:red;">结论:</strong> ImageNet数据集上略优于Inception V3；
> 
><strong style="color:red;">理解:</strong> 深度可分离卷积，Pointwith和Depthwise的先后关系，Xception是先用1x1卷积分组，然后每组进行3x3卷积但是大多实现还是3x3分组卷积+1x1点卷积，而第二点不同是点卷积输出是线性激活，没有使用激活函数，而是把激活函数调整到卷积前。
> 
><strong style="color:red;">优化：</strong>常规卷积和深度可分离卷积之间存在discrete spectrum，其参数是用于执行空间卷积的独立通道空间段的数量。初始模块是这一范围的重点。作者在经验评估中表明，与常规的Inception模块相比，Inception模块的极端情况（深度可分离卷积）可能具有优势。但是，没有理由相信深度可分离卷积是最佳的。可能是discrete spectrum上的中间点位于常规的Inception模块与深度可分离的卷积之间，具有其他优势。这个问题留待将来调查。
---

## 背景知识

深度可分离卷积（Depthwise Separable Convolution）率先是由 Laurent Sifre在其博士论文《Rigid-Motion Scattering For Image Classification》中提出。经典的MobileNet系列算法便是采用深度可分离卷积作为其核心结构。

## 原理方法

Xception取义自Extreme Inception，即Xception是一种极端的Inception。

### 1、简化Inception

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213846.png width=35% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213846.png width=35% />
</div>





Inception的核心思想是将通道分成若干个不同感受野大小的通道，除了能获得不同的感受野，Inception还能大幅的降低参数数量。

以简化版本为例：对于一个输入的特征图，首先通过三组1x1卷积得到三组特征图，它和先使用一组1x1卷积得到特征图，再将这组特征图分成三组是完全等价的。假设1x1卷积核的个数都是k2， 3x3的卷积核的个数都是k2，输入特征图的通道数为m，那么这个简单版本的参数个数为：

$$
m \times k_1 \times 1 \times 1 + 3 \times 3 \times 3 \times \frac{k_1}{3} \times \frac{k_2}{3} = m \times k_1 + 3 \times k_1 \times k_2
$$

对比相同通道数，但是没有分组的普通卷积，普通卷积的参数数量为：
$$
m \times k_1 \times 1 \times 1 + 3 \times 3 \times k_1 \times k_2
$$

Inception与普通卷积的参数数量约为Inception的三倍。



### 2、Xception

Inception是将3x3卷积分成3组，考虑一种极端的情况，将Inception的1x1得到的k1个通道的特征图完全分开，也就是使用k1个不同的卷积分别在每个通道上进行卷积，它的参数数量是：

$$
m \times k_1 + k_1 \times 3 \times 3
$$
更多时候作者希望两组卷积的输出特征图相同，这里我们将Inception的1x1卷积的通道数设为k2，即参数数量为

$$
m \times k_2 + k_2 \times 3 \times 3
$$
它的参数数量是普通卷积的$\frac{1}{k1}$，作者把这种形式的Inception叫做Extreme Inception，使用1x1卷积来映射跨通道相关性，然后分别映射每个输出通道的空间相关性：

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213847.png width=75% />
</div>

### 3、Xception 网络结构

在搭建GoogLeNet网络时，作者一般采用堆叠Inception的形式，同理在搭建由Extreme Inception构成的网络的时候也是采用堆叠的方式，论文中将这种形式的网络结构叫做Xception。

深度可分离卷积等价于Xception，但存在两点不同：

1. 区别之一就是先计算Pointwise卷积和先计算Depthwise的卷积的区别。

2. 在MobileNet v2中，指出bottleneck的最后一层1x1卷积核为线性激活时能够更有助于减少信息损耗，这也就是Xception和深度可分离卷积（准确说是MobileNet v2）的第二个不同点。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213847.png width=75% />
</div>



## 训练测试

由于它们在规模上的相似性，我们选择将Xception与Inception V3架构进行比较：Xception和Inception V3具有几乎相同数量的参数，因此任何性能差距都不能归因于网络参数量的差异。我们对两个图像分类任务进行了比较：一个是ImageNet数据集上著名的1000类单标签分类任务，另一个是大规模JFT数据集上17000类多标签分类任务。

**JFT数据集**

JFT是用于大型图像分类数据集的内部Google数据集，其中包括超过3.5亿张高分辨率图像，这些图像带有来自17,000个类别的标签的注释。 为了评估在JFT上训练的模型的性能，我们使用了辅助数据集FastEval14k。
FastEval14k是14,000张图像的数据集，具有来自6,000个类别的密集注释（平均每张图像36.5个标签）。在此数据集上，我们使用mAP对前100个预测（MAP @100）进行评估，并对每个类别对MAP@100的贡献进行加权，并给出一个分数，以估算该类别在社交媒体图像中的普遍程度（因此很重要）。此评估程序旨在从社交媒体上捕获频繁出现的标签上的效果，这对于Google的生产模型至关重要。

**优化器配置**

ImageNet：Optimizer=SGD，Momentum=0.9，Initial learning rate=0.045，Learning rate decay=decay of rate 0.94 every 2 epochs
JFT：Optimizer=RMSprop，Momentum=0.9，Initial learning rate=0.001，Learning rate decay=decay of rate 0.9 every 3,000,000 samples

**正则化配置**

Weight decay：Inception v3为0.00004，Xception为0.00001
Dropout：ImageNet为0.5，JFT无，因为数据太多，不太可能过拟合
Auxiliary loss tower：没有使用

**训练配置**

所有网络均使用TensorFlow框架实施，并分别在60个NVIDIA K80 GPU上进行了培训。 对于ImageNet实验，我们使用具有同步梯度下降的数据并行性来获得最佳的分类性能，而对于JFT，我们使用异步梯度下降来加快训练速度。 ImageNet实验每个大约花费3天，而JFT实验每个大约花费一个月。 JFT模型没有经过完全收敛的训练，而每个实验将花费三个月以上的时间。

**与Inception V3相比**

1. 在分类性能上，Xception在ImageNet领先较小，但在JFT上领先很多。

2. 在参数量和速度，Xception参数量少于Inception，但速度更快。

3. 作者还比较了residual connections，有了性能更强；还有点卷积之后要不要激活函数，没有非线性层效果最好。

   

## 代码

注意：实验发现，深度可分离卷积中的卷积层之间不加非线性激活函数的效果相较于加入非线性激活函数来说会更好一些，而是先激活，再深度可分离卷积，再加上归一化操作，最后跳跃链接相加。

```python
#深度可分离卷积
class SeparableConv2d(nn.Module):
    def __init__(self,in_channels,out_channels,kernel_size,stride,padding,dilation=1,bias=False):
        super(SeparableConv2d,self).__init__()
        # 逐通道卷积：groups=in_channels=out_channels
        self.conv1 = nn.Conv2d(in_channels,in_channels,kernel_size,stride,padding,dilation,groups=in_channels,bias=bias)
        # 逐点卷积：普通1x1卷积
        self.pointwise = nn.Conv2d(in_channels,out_channels,kernel_size=1,stride=1,padding=0,dilation=1,groups=1,bias=bias)
    
    def forward(self,x):
        x = self.conv1(x)
        x = self.pointwise(x)
        return x
```



## 参考文献

[^01]: [咫尺小厘米-[论文笔记] Xception-知乎](https://zhuanlan.zhihu.com/p/127042277)

[^02]: [大师兄-Xception详解-知乎](https://zhuanlan.zhihu.com/p/50897945)


