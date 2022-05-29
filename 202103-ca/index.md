# Coordinate Attention for Efficient Mobile Network Design

摘要：CA通过2D全局池将特征张量转换为单个特征向量的通道关注不同，**坐标关注将通道关注分解为两个1D特征编码过程，分别沿两个空间方向聚合特征。** 这样，可以沿一个空间方向捕获远程依赖关系，同时可以沿另一空间方向保留精确的位置信息。然后将生成的特征图分别编码为一对方向感知和位置敏感的注意图，可以将其互补地应用于输入特征图，以增强关注对象的表示。
<!--more-->

# Coordinate Attention for Efficient Mobile Network Design[^01]

## 文献信息
| 信息 | 内容                                                         |
| ---- | ------------------------------------------------------------ |
| 日期 | 2021.04                                                      |
| 作者 | Qibin Hou et al.                                             |
| 机构 | National University of Singapore                             |
| 来源 | cvpr2021                                                     |
| 链接 | [Coordinate Attention for Efficient Mobile Network Design](http://arxiv.org/abs/2103.02907) |
| 代码 | [Andrew-Qibin/CoordAttention](https://github.com/Andrew-Qibin/CoordAttention) |

## 个人理解

><strong style="color:red;">问题:</strong> 注意力机制问题；
>
><strong style="color:red;">方法:</strong> 分离x和y方向，分别计算注意力机制，构建Coordinate Attention；
>
><strong style="color:red;">结论:</strong> 协调注意不仅有利于ImageNet分类，而且更有趣的是，它在下游任务中表现得更好，例如目标检测和语义分割；
>
><strong style="color:red;">理解:</strong> 
>
>1. 分离x和y方向，计算平均池化；
>2. 拼接 + 卷积(通道下采样) + BN + 激活函数；
>3. 分离 + 分别卷积（通道上采样）+ 分别sigmod；
>4. Re-weight；
>5. 思想：允许注意力模块捕捉到**沿着一个空间方向的长期依赖关系，并保存沿着另一个空间方向的精确位置信息(通过通道来保另一个空间的精确位置信息)**，这有助于网络更准确地定位感兴趣的目标。
>
><strong style="color:red;">优化：</strong>无。
---

## 背景知识

SENet：基于通道注意力机制，简单地squeeze每个2维特征图，进而有效地构建通道之间的相互依赖关系。r是用来控制SE block大小的缩减率。

CBAM：基于通道和空间信息的注意力机制。类似的，GENet、GALA、AA、TA，通过采用不同的空间注意力机制或设计高级注意力块，扩展了这一理念。

Non-local/Self-attention Network：

1. 根据各像素之间的相关性，对所有像素进行加权。权重越大，说明这个区域越重要。

2. NLNet、GCNet、A2Net、SCNet、GsopNet和CCNet，利用Non-local机制来捕获不同类型的空间信息。

3. 由于Self-attention模块内部计算量大，常被用于大型模型中，不适用于Mobile Network。

## 原理方法

### 1、Coordinate Attention
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220510101546974.png width=75% />
</div>


全局池化：用于通道注意编码空间信息的全局编码，但由于它将全局空间信息压缩到通道描述符中，导致难以保存位置信息。

$$
z_{c}=\frac{1}{H \times W} \sum_{i=1}^{H} \sum_{j=1}^{W} x_{c}(i, j)
$$


Coordinate信息嵌入：使用尺寸为(H,1)或(1,W)的pooling kernel分别沿着水平坐标和垂直坐标对每个通道进行编码。两个空间方向聚合特征，得到一对方向感知的特征图。

$$
\begin{aligned} z_{c}^{h}(h) &=\frac{1}{W} \sum_{0 \leq i<W} x_{c}(h, i) \\ z_{c}^{w}(w) &=\frac{1}{H} \sum_{0 \leq j<H} x_{c}(j, w) \end{aligned}
$$
特点：允许注意力模块捕捉到沿着一个空间方向的长期依赖关系，并保存沿着另一个空间方向的精确位置信息，这有助于网络更准确地定位感兴趣的目标。

### 2、Coordinate Attention生成

设计原则：

- 首先，对于Mobile环境中的应用来说，新的转换应该尽可能地简单。
- 其次，它可以充分利用捕获到的位置信息，使感兴趣的区域能够被准确地捕获。
- 最后，它还应该能够有效地捕捉通道间的关系。


步骤：

1. Concat + conv2d。

$$
{\bf{f}} = \delta \left( {{F_{conv - 1 \times 1}}\left( {\left[ {{{\bf{z}}^h},{{\bf{z}}^w}} \right]} \right)} \right)
$$



2. BatchNorm + Non-linear。

3. Spilt+Conv2d+sigmoid。


$$
{{\bf{g}}^h} = \sigma \left( {{F_{h - Conv1 \times 1}}\left( {{{\bf{f}}^h}} \right)} \right)
$$

$$
{{\bf{g}}^w} = \sigma \left( {{F_{w - Conv1 \times 1}}\left( {{{\bf{f}}^w}} \right)} \right)
$$



4. Re-weight。

$$
{y_c}(i,j) = {x_c}(i,j) \times g_c^h(i) \times g_c^w(j)
$$

### 3、CA嵌入方式
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220510101659847.png width=75% />
</div>

## 实验结果

不同注意力机制对比实验结果：
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220510101751575.png width=75% />
</div>

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220510101727043.png width=75% />
</div>

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220510101821309.png width=75% />
</div>


<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220510101827964.png width=75% />
</div>

可视化不同注意力机制实验结果：
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220510101926904.png width=75% />
</div>


嵌入不同网络实验结果：
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220510101846144.png width=75% />
</div>


目标检测实验结果：
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220510101959065.png width=75% />
</div>

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220510102012308.png width=75% />
</div>


图像分割实验结果：
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220510102030313.png width=75% />
</div>

<div align=center>
    <img https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220510102039588.png width=75% />
</div>


## 参考文献

[^01]: [陋室了凡-CVPR2021-即插即用 | Coordinate Attention详解与CA Block实现(文末获取论文原文)-oschina](https://my.oschina.net/u/3776677/blog/4976696)


