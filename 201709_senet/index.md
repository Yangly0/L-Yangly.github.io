# Squeeze-and-Excitation Networks

摘要：SENet引进通道注意机制，它赢得了最后一届ImageNet 2017竞赛分类任务的冠军。
<!--more-->

## 文献信息

| 信息 | 内容                                                         |
| ---- | ------------------------------------------------------------ |
| 日期 | 2017.09                                                      |
| 作者 | Jie Hu et al.                                                |
| 机构 | momenta                                                      |
| 来源 | CVPR2017                                                     |
| 链接 | [Squeeze-and-Excitation Networks](https://arxiv.org/abs/1709.01507) |
| 代码 | [hujie-frank/SENet](https://github.com/hujie-frank/SENet)、[moskomule/senet.pytorch](https://github.com/moskomule/senet.pytorch) |

## 个人理解
><strong style="color:red;">问题:</strong> 通道注意力机制；
>
><strong style="color:red;">方法:</strong> 全局平均池化 + 2层缩放全连接 + sigmod + 权重相乘；
>
><strong style="color:red;">结论:</strong> 2017年ILSVRC分类提交的基础，获得了第一名，并将前五名的误差降低到2:251%，比2016年的获奖作品相对提高了25%；
>
><strong style="color:red;">理解:</strong> 通道注意力机制，让模型可以更加关注信息量最大的channel特征，而抑制那些不重要的channel特征。
>
><strong style="color:red;">优化：</strong>无。
---

## 背景知识

**卷积神经网络本质：**卷积核从输入特征图学习到新特征图，即一个局部区域进行特征融合，包括空间（H和W维度）以及通道间（C维度）。

- 空间维度层面：（1）感受野，即空间上融合更多特征融合或者多尺度空间信息，如Inception网络的多分支结构；（2）在 Inside-Outside 网络中考虑了空间中的上下文信息；（3）Attention 机制引入空间维度；
- 通道维度层面：对输入特征图的所有channel进行融合；通道分离（组卷积和深度可分离卷积）。

## 原理方法

### 1、Squeeze-and-Excitation (SE) 模块

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213906.png width=75% />
</div>


SE模块主要包括Squeeze和Excitation两个操作，可以适用于任何映射$ F_{t r}: X \rightarrow U, X \in R^{H^{\prime} \times W^{\prime} \times C^{\prime}}, U \in R^{H \times W \times C} $, 以卷积为例，卷积核为:$ V=\left[v_{1}, v_{2}, \ldots, v_{c}\right]$，其中$v_c$ 表示第c个卷积核，那么输出 $ U=\left[u_{1}, u_{2}, \ldots, u_{C}\right] $
$$
u_{c}=v_{c} X=\sum_{s=1}^{C^{\prime}} v_{c}^{s} x^{s}
$$
其中*代表卷积操作，而$v_s^c$ 代表一个3D卷积核，其输入一个channel上的空间特征，它学习特征空间关系，但是由于对各个channel的卷积结果做了sum，所以channel特征关系与卷积核学习到的空间关系混合在一起。而SE模块就是为了抽离这种混杂，使得模型直接学习到channel特征关系。

- Squeeze操作


由于卷积只是在一个局部空间内进行操作，$U$很难获得足够的信息来提取channel之间的关系，对于网络中前面的层这更严重，因为感受野比较小。**SENet提出Squeeze操作，将一个channel上整个空间特征编码为一个全局特征，采用global average pooling来实现（原则上也可以采用更复杂的聚合策略）**：

$$
z_{c}=F_{s q}\left(u_{c}\right)=\frac{1}{H \times W} \sum_{i=1}^{H} \sum_{j=1}^{W} u_{c}(i, j), z \in R^{C}
$$

- Excitation操作

queeze操作得到了全局描述特征，然后需要抓取channel之间的关系。这个操作需要满足两个准则：首先要灵活，它要可以学习到各个channel之间的非线性关系；第二点是学习的关系不是互斥的，因为这里允许多channel特征，而不是one-hot形式。基于此，这里采用sigmoid形式的gating机制：

$$
s=F_{e x}(z, W)=\sigma(g(z, W))=\sigma\left(W_{2} \operatorname{ReLU}\left(W_{1} z\right)\right)
$$
其中$W_{1} \in R^{\frac{C}{r} \times C}, W_{2} \in R^{C \times \frac{C}{r}}$，**为了降低模型复杂度以及提升泛化能力，采用包含两个全连接层的bottleneck结构，其中第一个FC层起到降维的作用，降维系数为r是个超参数，然后采用ReLU激活。最后的FC层恢复原始的维度。**

**最后将学习到的各个channel的激活值（sigmoid激活，值0~1）乘以U上的原始特征：**
$$
\tilde{x} c=F \operatorname{scale}\left(u_{c}, s_{c}\right)=s_{c} \cdot u_{c}
$$
其实整个操作可以看成学习到了各个channel的权重系数，从而使得模型对各个channel的特征更有辨别能力，这应该也算一种attention机制。

### 3、SENet 网络结构
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213907.png width=75% />
</div>


注意：增加了SE模块后，模型参数以及计算量都会增加，这里以SE-ResNet-50为例，对于模型参数增加量为：

$$
\frac{2}{r} \sum_{s=1}^{S} N_{s} \cdot C_{s}^{2}
$$

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213907.png width=75% />
</div>


不同的嵌入方式：

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213908.png width=75% />
</div>




## 实验结果

嵌入流行网络：

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213908.png width=75% />
</div>


嵌入轻量级网络：

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213909.png width=75% />
</div>


在图像分类和目标检测领域结果：

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213910.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213911.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213911.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213912.png width=75% />
</div>


<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220509093717714.png width=75% />
</div>

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220509093733137.png width=75% />
</div>


<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220509093742797.png width=75% />
</div>


## 参考文献

[^01]: [小小将-最后一届ImageNet冠军模型：SENet-知乎](https://zhuanlan.zhihu.com/p/65459972/)






