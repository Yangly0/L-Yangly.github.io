# Densely Connected Convolutional Networks

摘要：DenseNet脱离了加深网络层数(ResNet)和加宽网络结构(Inception)来提升网络性能的定式思维，从特征的角度考虑，通过特征重用和旁路(Bypass)设置,既大幅度减少了网络的参数量，又在一定程度上缓解了gradient vanishing问题的产生。结合信息流和特征复用的假设，DenseNet当之无愧成为2017年计算机视觉顶会的年度最佳论文。
<!--more-->

# Densely Connected Convolutional Networks[^01]

## 文献信息
| 信息 | 内容                                                         |
| ---- | ------------------------------------------------------------ |
| 日期 | 2016.08                                                      |
| 作者 | Gao Huang, Zhuang Liu, Laurens van der Maaten, Kilian Q. Weinberger. |
| 机构 | Cornell University, Tsinghua University, Facebook AI Research, Cornell University |
| 来源 | CVPR 2017 年度最佳论文                                       |
| 链接 | [Densely Connected Convolutional Networks](https://arxiv.org/abs/1608.06993) |
| 代码 | [liuzhuang13/DenseNet](https://github.com/liuzhuang13/DenseNet) |

## 个人理解

><strong style="color:red;">问题:</strong> 文章为了解决梯度消失和参数问题；
>
><strong style="color:red;">方法:</strong> 文章提出了信息流和特征复用思想，即cat而不是add，保证信息流的流通，特征复用是前面所有层cat，这样可用用少量的通道组成大通道数；
>
><strong style="color:red;">结论:</strong> CIFAR-10、CIFAR-100、SVHN和ImageNet 达到SOTA；
>
><strong style="color:red;">理解:</strong> 
>
>1. 信息流，更强的梯度流动：DenseNet可以说是一种隐式的强监督模式，因为每一层都建立起了与前面层的连接，误差信号可以很容易地传播到较早的层，所以较早的层可以从最终分类层获得直接监管。
>2. 参数更少计算效率更高：在ResNet中，参数量与$C \times C$成正比，而在DenseNet中参数量与$l \times k \times k $成正比，因为k远小于C，所以DenseNet的参数量小得多。
>3. 保存了低维度的特征：在标准的卷积网络中，最终输出只会利用提取最高层次的特征。而在DenseNet中，它使用了不同层次的特征,它倾向于给出更平滑的决策边界。这也解释了为什么训练数据不足时DenseNet表现依旧良好。
>
><strong style="color:red;">优化：</strong>但是个人感觉DensNet没有ResNet用得广？不知道什么原因。
---

## 背景知识

随着CNN网络层数的不断增加，gradient vanishing和model degradation问题出现在了人们面前，BatchNormalization的广泛使用在一定程度上缓解了gradient vanishing的问题，而ResNet和Highway Networks通过构造恒等映射设置旁路，进一步减少了gradient vanishing和model degradation的产生。Fractal Nets通过将不同深度的网络并行化,在获得了深度的同时保证了梯度的传播，随机深度网络通过对网络中一些层进行失活,既证明了ResNet深度的冗余性，又缓解了上述问题的产生。虽然这些不同的网络框架通过不同的实现加深的网络层数，但是他们都包含了相同的核心思想，既将特征图进行跨网络层的连接。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/v2-e1fa79bab2f9407c18858aa46b94bc94_r.jpg width=75% />
</div>



DenseNet作为另一种拥有较深层数的卷积神经网络，具有如下优点:

(1) 相比ResNet拥有更少的参数数量。

(2) 旁路加强了特征的重用。

(3) 网络更易于训练，并具有一定的正则效果。

(4) 缓解了gradient vanishing和model degradation的问题。

何恺明先生在提出ResNet时做出了这样的假设：若某一较深的网络多出另一较浅网络的若干层有能力学习到恒等映射，那么这一较深网络训练得到的模型性能一定不会弱于该浅层网络。通俗的说就是如果对某一网络中增添一些可以学到恒等映射的层组成新的网路，那么最差的结果也是新网络中的这些层在训练后成为恒等映射而不会影响原网络的性能。

同样DenseNet在提出时也做过假设:与其多次学习冗余的特征,特征复用是一种更好的特征提取方式。

## 原理方法

假设输入为一个图片$X_0$ , 经过一个L层的神经网络, 其中第i层的非线性变换记为$H_i(*)$, $H_i(*)$可以是多种函数操作的累加如BN、ReLU、Pooling或Conv等. 第i层的特征输出记作$X_i$。

ResNet：传统卷积前馈神经网络将第i层的输出$X_i$作为i+1层的输入,可以写作$X_i = H_i(X_{i-1})$. ResNet增加了旁路连接,可以写作

$$
X_i = H_i(X_{i-1}) + X_{i-1}
$$

ResNet的一个最主要的优势便是梯度可以流经恒等函数来到达靠前的层。但恒等映射和非线性变换输出的叠加方式是相加, 这在一定程度上破坏了网络中的信息流。

### 1、 密集连接

为了进一步优化信息流的传播，DenseNet提出了图示的网络结构，

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220428235723063.png width=75% />
</div>

如图所示，第i层的输入不仅与i-1层的输出相关，还有所有之前层的输出有关。记作:

$$
	X_l = H_l ([X_0, X_1, ..., X_{l-1}])
$$

其中[]代表concatenation(拼接),既将$X_1$ 到$X_{l-1}$层的所有输出特征图按通道组合在一起。这里所用到的非线性变换H为`BN+ReLU+ Conv(3×3)`的组合。

### 2、池化层

在DenseNet中需要对不同层的特征图进行cat操作，所以需要不同层的特征图保持相同的特征尺度，这就限制了网络中下采样的实现。为了使用下采样，作者将DenseNet分为多个Denseblock:


<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220429000814433.png width=75% />
</div>

在同一个Denseblock中要求特征尺度保持相同大小，在不同Denseblock之间设置过渡层（transitionlayer）实现Downsampling, 在作者的实验中过渡层（transitionlayer）由`BN + Conv(1×1) ＋2×2 average-pooling`组成。

### 3、 增长率 Growth rate

在Denseblock中，假设每一个非线性变换H的输出为K个特征图, 那么第i层网络的输入便为$K_0 + (i-1)×K$, 但DenseNet不同点是可以接受较少的特征图数量作为网络层的841输出，

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220429000941071.png width=75% />
</div>

原因就是在同一个Denseblock中的每一层都与之前所有层相关联，如果把feature看作是一个Denseblock的全局状态，那么每一层的训练目标便是通过现有的全局状态，判断需要添加给全局状态的更新值。因而每个网络层输出的特征图数量K又称为Growth rate，同样决定着每一层需要给全局状态更新的信息的多少。在作者的实验中只需要较小的K便足以实现state-of-art的性能。

### 4、瓶颈层 Bottleneck Layers

虽然DenseNet接受较少的k，即特征图的数量作为输出，但由于不同层特征图之间由cat操作组合在一起，最终仍然会是特征图的通道较大而成为网络的负担。作者使用`1×1 Conv(Bottleneck)`作为特征降维的方法来降低通道数量，以提高计算效率。经过改善后的非线性变换变为`BN-ReLU-Conv(1×1)-BN-ReLU-Conv(3×3)`,使用Bottleneck layers的DenseNet被作者称为DenseNet-B。在实验中，作者使用1×1卷积生成通道数量为4k的特征图。

### 5、压缩 Compression：

为了进一步优化模型的简洁性，同样可以在过渡层transition layer中降低特征图的数量。若一个Denseblock中包含m个特征图s，那么使其输出连接的transition layer层生成⌊θm⌋个输出特征图。其中θ为Compression factor, 当θ=1时，过渡层transition layer将保留原feature维度不变。

作者将使用compression且θ=0.5的DenseNet命名为DenseNet-C, 将使用Bottleneck和compression且θ=0.5的DenseNet命名为DenseNet-BC。

## 实验结果

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220429001127339.png width=75% />
</div>
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220429001215361.png width=75% />
</div>
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220429001236521.png width=75% />
</div>


## 参考文献

[^01]: [SIGAI-DenseNet详解-知乎](https://zhuanlan.zhihu.com/p/43057737)


