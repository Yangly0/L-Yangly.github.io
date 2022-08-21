# Inception-v4, Inception-ResNet and the Impact of Residual Connections on Learning

摘要：Incetpion网络趋于深度化，提高网络容量的同时还能保证计算复杂性不至于过高，而ResNet网络解决了在网络深度化时准确率下降的问题，所以很自然地就想到将这两者结合起来，组成Inception-ResNet。
<!--more-->

# Inception-v4, Inception-ResNet and the Impact of Residual Connections on Learning[^01]

## 文献信息
| 信息               | 内容                                                                                                           |
| ------------------ | ----------------------------------------------------------------------------------------------------------------- |
| 日期               | 2016.02                                                                                                      |
| 作者               | Christian Szegedy et al.                                                                                                        |
| 机构               | Google Inc                                                                                                          |
| 来源               | arXiv                                                                                                           |
| 链接               | [Inception-v4, Inception-ResNet and the Impact of Residual Connections on Learning](https://arxiv.org/abs/1602.07261)                                                                                                         |
| 代码               | [Code]()                                                                                                          |

## 个人理解
>
> <strong style="color:red;">问题:</strong> 针对Inception和ResNet网络结构，如何优化；
>
> <strong style="color:red;">方法:</strong> 文章提出了Inception-V4以及与ResNet结合的Inception-ResNet-V2结构，并验证了残差并不是必要条件，只是加快了训练速度；
>
> <strong style="color:red;">结论:</strong> ILSVRC 2012，top-5错误率上达到了 3.08%；
>
> <strong style="color:red;">理解:</strong> 论文发现不使用Residual connection也可以训练更深的网络。Residual connection并不是必要条件，只是使用了Residual connection会加快训练速度。
>
> <strong style="color:red;">优化：</strong>略。
---

## 背景知识
1. 噪声标签的意思，参考[深度学习中噪声标签的影响和识别](https://zhuanlan.zhihu.com/p/28703943)，指去掉噪声标签剩下的都是true label，不能识别所有的true label，所以有提升空间。[^01]

2. 论文对ResNet加深网络的观点不是很同意。	

## 原理方法

### 1、Pure Inception blocks

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427164902571.png width=50% />
</div>



### 2、Residual Inception Blocks

不同残差结构：

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427165239343.png width=25% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427165255911.png width=50% />
</div>

Inception-ResNet 网络结构：

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427165109777.png width=75% />
</div>

### 3、 Scaling of the Residuals

加宽网络有时会难以训练，解决方法是缩放residual项。当过滤器的数目超过1000个的时候，会出现问题，网络会“坏死”，即在average pooling层前都变成0。即使降低学习率，增加BN层都没有用，这时候就在激活前缩小残差可以保持稳定。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427165543504.png width=75% />
</div>

## 实验结果

网络精度提高原因：残差连接只能加速网络收敛，真正提高网络精度的还是“更大的网络规模”。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427165629443.png width=50% />
	<img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427165646911.png width=50% />
</div>

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427165659837.png width=50% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427165716644.png width=50% />
</div>

## 参考文献

[^01]:[Michael-深入解读Inception V4（附源码）-知乎](https://zhuanlan.zhihu.com/p/49597731)


