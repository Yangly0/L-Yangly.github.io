# EfficientNet, Rethinking Model Scaling for Convolutional Neural Networks

摘要：EfficientNet提出了一种新的模型缩放方法，它使用一个简单而高效的复合系数来从depth, width, resolution 三个维度放大网络，不会像传统的方法那样任意缩放网络的维度，基于神经结构搜索技术可以获得最优的一组参数(复合系数)。
<!--more-->

# EfficientNet: Rethinking Model Scaling for Convolutional Neural Networks

## 文献信息

| 信息 | 内容                                                         |
| ---- | ------------------------------------------------------------ |
| 日期 | xx, xx xxxx                                                  |
| 作者 | Mingxing Tan et al.                                                   |
| 机构 | xxxxxxxx                                                     |
| 来源 | 期刊杂志                                                     |
| 链接 | [EfficientNet: Rethinking Model Scaling for Convolutional Neural Networks](https://arxiv.org/abs/1905.11946) |
| 代码 | [Code]()                                                     |

## 个人理解
>
> <strong style="color:red;">问题:</strong> 文章为了解决什么问题；
>
> <strong style="color:red;">方法:</strong> 文章提出了什么方法和技术；
>
> <strong style="color:red;">结论:</strong> 文章结论即 数据集 + 评价 指标；
>
> <strong style="color:red;">理解:</strong> 论文读完的感受与体会，比如该方法借鉴了什么思想，方法是不是新颖，实验怎么做的，讨论的变量是什么，还有其他值得读的文献。
>
> <strong style="color:red;">优化：</strong>还有什么值得改进与优化的。
---

## 背景知识

卷积神经网络（ConvNets）通常是在固定的资源预算下发展起来的，如果有更多的资源可用的话，则会扩大规模以获得更好的精度，比如可以提高网络深度(depth)、网络宽度(width)和输入图像分辨率 (resolution)大小。但是通过人工去调整 depth, width, resolution 的放大或缩小的很困难的，在计算量受限时有放大哪个缩小哪个，这些都是很难去确定的，换句话说，这样的组合空间太大，人力无法穷举。

## 原理方法

### 1、网络参数讨论

问题定义：我们将整个卷积网络称为 N，它的第 i 个卷积层可以表示为：$ Y_{i}=F_{i}\left(X_{i}\right)$, $X_{i} $ 代表输入张量,$ \quad Y_{i} $ 代表输出张量。整个卷积网络由 k 个卷积层组成，可以表示为：

$$
\mathcal{N}=\mathcal{F}_{k} \odot \ldots \odot \mathcal{F}_{2} \odot \mathcal{F}_{1}\left(X_{1}\right)=\odot_{j=1 \ldots k} \mathcal{F}_{j}\left(X_{1}\right)
$$

但是在实际中，通常会将多个结构相同的卷积层称为一个stage，例如ResNet有5个stage，每个stage中的卷积层结构相同(除了第一层为降采样层)，以 stage 为单位可以将卷积网络 N 表示为：

$$
\mathcal{N}=\bigoplus_{i=1 \ldots s} \mathcal{F}_{i}^{L_{i}}\left(X_{\left\langle H_{i}, W_{i}, C_{i}\right\rangle}\right)
$$

其中，$ <H_{I}, W_{I}, C_{i}> $代表第i层的输入张量的维度（为了方便叙述忽略 batch 这个维度），下标 i(从 1 到 s) 表示的是 stage 的序号，$ F_{i}^{L_{i}} $表示第 i 个 stage ，它由卷积层$F_i$重复$L_i$次构成。

与通常的ConvNet设计不同，通常的ConvNet设计主要关注寻找最佳的网络层$F_i$, 模型缩放尝试扩展网络长度（$L_i$）、宽度（$C_i$）和分辨率（$(H_i,W_i)$），而不改变基线网络中预定义的$F_i$（个人在这里的理解是指kernel size等每一个层内的参数，因为模型缩放只对depth, width, resolution进行组合调整，不对每一个层内具体的方式做改变）。

所以，优化目标就是在资源有限的情况下，要最大化 Accuracy, 优化目标的公式表达如下：

$$
\max_{d, w, r}  \text { Accuracy }(\mathcal{N}(d, w, r)) \\
\text { s.t. }  \mathcal{N}(d, w, r)=\underset{i=1 \ldots s}{\bigoplus} \hat{\mathcal{F}}_{i}^{d \cdot \hat{L}_{i}}\left(X_{\left\langle r \cdot \hat{H}_{i}, r \cdot \hat{W}_{i}, w \cdot \hat{C}_{i}\right\rangle}\right)     \\
\operatorname{Memory}(\mathcal{N}) \leq \text { target\_memory } \\
\operatorname{FLOPS}(\mathcal{N}) \leq \text { target\_flops }  \\
$$


作者发现，更大的网络具有更大的宽度、深度或分辨率，往往可以获得更高的精度，但精度增益在达到80%后会迅速饱和，这表明了只对单一维度进行扩张的局限性，实验结果如下：



作者指出，模型扩张的各个维度之间并不是完全独立的，比如说，对于更大的分辨率图像，应该使用更深、更宽的网络，这就意味着需要平衡各个扩张维度，而不是在单一维度张扩张。



直线上的每个点表示具有不同宽度系数（w）的模型：第一个基线网络（d=1.0，r=1.0）有18个卷积层，分辨率224x224，而最后一个基线（d=2.0，r=1.3）有36个卷积层，分辨率299x299。

这个图说明了一个问题，为了追求更好的精度和效率，在ConvNet缩放过程中平衡网络宽度、深度和分辨率的所有维度是至关重要的。

### 2、复合模型扩张方法

本文提出了复合扩张方法，组合深度、宽度和分辨率，即$(\alpha, \beta, \gamma)$是求解的一组参数，带约束的最优参数求解。$(\alpha, \beta, \gamma)$分别衡量着depth, width和resolution的比重，其中 $\beta,\gamma$在约束上会有平方，是因为增加宽度或分辨率两倍，对应计算量是增加四倍，但是增加深度两倍，其计算量只会增加两倍。

$$
depth:  d=\alpha^{\phi} \\
width:  w=\beta^{\phi}   \\
resolution:  r=\gamma^{\phi} \\
s.t. \alpha \cdot \beta^{2} \cdot \gamma^{2} \approx 2 \\
\alpha \geq 1, \beta \geq 1, \gamma \geq 1
$$



迭代求解方式：

1. 固定公式中的$φ=1$，然后通过网格搜索（grid search）得出最优的α、β、γ，得出最基本模型EfficientNet-B0;
2. 固定$α, β, γ$的值，使用不同的$φ$，得到EfficientNet-B1, ..., EfficientNet-B7;

$φ$的大小对应着消耗资源的大小，相当于：

1. 当φ=1时，得出了一个最小的最优基础模型；
2. 增大φ时，相当于对基模型三个维度同时扩展，模型变大，性能也会提升，资源消耗也变大;

对于神经网络搜索，作者使用了和 `MnasNet: Platform-awareneural architecture search for mobile` 一样的搜索空间和优化目标。

第一处区别是在最开始降采样没有采用maxpooling，而是换成了stride为2的conv。。猜测是为了减少信息丢失，尤其是对小模型来说，前期的底层特征提取更重要。

第二处区别是第一次降采样后的channel反而减少了，这个我没搞懂。。。。

第三处区别是有很多stage都采用了5x5的conv。。。这是因为对于depthwise separable conv来说，5x5的计算量要比两个3x3的计算量要小。。（坊间传闻large kernel is all your need. 233333）

[公式]

其中输入特征图尺寸为(H, W, M)，输出特征图尺寸为(H, W, N)。

第四处区别是降采样后的特征图尺寸减半，但是channel没有扩大两倍。第6个stage特征图尺寸没变，但是channel也扩大了。。这些可能都是手工设计很难搞定的。。。我能想到的解释是，MnasNet在搜网络结构的时候带上了运算量的约束，可以理解成网络在训练的时候就考虑了pruing(裁枝)，因此才会出现一些不规则的channel数，同时这个带来另外的一个好处就是，网络可以更好的训练更有意义的权重，因此这些搜出来的网络结构的上限更高。



MBConv



，发现实际部署算法的时候往往不太追求极限精度，反而更多的要考虑模型大小和资源消耗



而EfficientNet用复合scaling的方法告诉你，用他这种方法，从 精度低的小网络 放缩到 精度高的大网络 ，精度和资源消耗能呈现出正相关的效果



**small** grid search

## 训练测试

## 参考文献

[^01]: [令人拍案叫绝的EfficientNet和EfficientDet-MoonSmile-知乎](https://zhuanlan.zhihu.com/p/96773680)


