# Learning Transferable Architectures for Scalable Image Recognition

摘要：cvpr2017 google brain作品，利用强化学习，使用500块p100训练4天多得到的网络结构NASNet，在小数据（CIFAR-10）上学习一个网络单元（Cell），然后通过堆叠更多的这些网络单元的形式将网络迁移到更复杂，尺寸更大的数据集上面，不管在精度还是在速度上都超越了人工设计的经典结构。
<!--more-->

# Learning Transferable Architectures for Scalable Image Recognition[^01][^02]

## 文献信息

| 信息 | 内容                                                         |
| ---- | ------------------------------------------------------------ |
| 日期 | 2017.07                                                      |
| 作者 | Barret Zoph et al.                                           |
| 机构 | Google Brain                                                 |
| 来源 | cvpr2018                                                     |
| 链接 | [Learning Transferable Architectures for Scalable Image Recognition](https://arxiv.org/abs/1707.07012) |
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

经典CNN结构如VGGNet、ResNet、DenseNet、MobileNet等等都是人工设计的，人工难免无法做到最优，既然深度学习这么强大，能否让CNN自己去设计CNN?


Google Brain在ICLR 2017最早提出NAS：

> Zoph B, Le Q V. Neural architecture search with reinforcement learning[J]. arXiv preprint arXiv:1611.01578, 2016.

CNN的结构可以由RNN产生，生成的CNN在目标数据集上训练收敛后会得到accuracy，以accuracy作为监督信号，用RL (Reinforcement Learning)训练RNN控制器，这个过程不断循环，就能生产越来越好的CNN结构。

CVPR2018上NASNet:

> Zoph B, Vasudevan V, Shlens J, et al. Learning transferable architectures for scalable image recognition[C]//Proceedings of the IEEE conference on computer vision and pattern recognition. 2018: 8697-8710.

ECCV 2018的PNAS：

> Liu C, Zoph B, Neumann M, et al. Progressive neural architecture search[C]//Proceedings of the European Conference on Computer Vision (ECCV). 2018: 19-34.

ENAS：

> Pham H, Guan M Y, Zoph B, et al. Efficient neural architecture search via parameter sharing[J]. arXiv preprint arXiv:1802.03268, 2018.

AAAI 2019的AmeobaNet-A：

>Real E, Aggarwal A, Huang Y, et al. Regularized evolution for image classifier architecture search[C]//Proceedings of the AAAI Conference on Artificial Intelligence. 2019, 33: 4780-4789.

Hardware-aware efficient NAS:

既然搜索来的block或许轻量但并不高效，第二类方法固定block结构转而搜索结构超参数，目前主流方法以MobileNetV2作为起点搜索，block用MBConv并保持其结构不动，只搜索stage和block相关的超参数，其中stage相关的超参数包括每个stage的block num和channel num，block相关的超参数包括kernel size, expansion factor, se ratio等。

Hardware-aware efficient NAS又可以分两个小类：

- 第一小类是RNN作为controller用RL训练的NAS，特点是搜索空间大但搜索时间很长，需要有强大计算资源支撑，代表模型MnasNet；
- 第二小类叫differentialble NAS，常用方法是训练一个SuperNet并从中pruning出小网络的方法，特征是搜索空间比较小但搜索速度非常快，搜索时间与MnasNet相比，ICLR 2019的ProxylessNAS快200倍，CVPR 2019的FBNet快420倍，Single-Path NAS快5000倍。

搜索FPN结构，比如CVPR 2019的NAS-FPN：

> Ghiasi G, Lin T Y, Le Q V. Nas-fpn: Learning scalable feature pyramid architecture for object detection[C]//Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition. 2019: 7036-7045.

搜索图像分类的augmentation，比如CVPR 2019的AutoAugment：

> Cubuk E D, Zoph B, Mane D, et al. AutoAugment: Learning Augmentation Strategies From Data[C]//Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition. 2019: 113-123.

搜索目标检测的augmentation，Learning Data Augmentation Strategies：

> Zoph B, Cubuk E D, Ghiasi G, et al. Learning Data Augmentation Strategies for Object Detection[J]. arXiv preprint arXiv:1906.11172, 2019.

搜索Deeplab中的ASPP，如NeurIPS 2018的DPC：

>Chen L C, Collins M, Zhu Y, et al. Searching for efficient multi-scale architectures for dense image prediction[C]//Advances in Neural Information Processing Systems. 2018: 8699-8710.

将Cell-basd NAS和ASPP一起搜索，CVPR 2019的Auto-DeepLab：

>Liu C, Chen L C, Schroff F, et al. Auto-deeplab: Hierarchical neural architecture search for semantic image segmentation[C]//Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition. 2019: 82-92.

搜索激活函数，如Swish，搜出来的结构是$x \times \alpha \times sigmod(\beta x)$，用于近期的MobileNetV3：

>Ramachandran P, Zoph B, Le Q V. Searching for activation functions[J]. arXiv preprint arXiv:1710.05941, 2017.

搜索优化方法，如Neural Optimizer Search，搜索出AddSign和PowerSign：

>Bello I, Zoph B, Vasudevan V, et al. Neural optimizer search with reinforcement learning[C]//Proceedings of the 34th International Conference on Machine Learning-Volume 70. JMLR. org, 2017: 459-468.

## 原理方法

### 1、NASNet 搜索过程
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220507233511566.png width=75% />
</div>

控制器RNN：从搜索空间中以概率p预测网络结构A。

worker单元：以结构A构建子网络，学习该网络直到收敛，并得到准确性R。

参数更新：最终将梯度p*R传递给RNN控制器进行梯度更新。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220507233800251.png width=75% />
</div>


控制器依次选择哪些Feature Map作为输入（灰色部分）以及使用哪些运算（黄色部分）来计算输入的Feature Map。其中，每种方法，每种操作都对应于一个softmax损失。这样重复B次，得到一个最终block模块。最终的损失函数就有5B个，实验中最优的B=5。

1. 从第$h_{i-1}$个Feature Map或者第 $h_{i}$个Feature Map或者已生成的网络块中选择一个Feature Map作为hidden layer A的输入，从上图中可以看到三种不同输入Feature Map的情况；
2. 采用和1类似的方法为Hidden Layer B选择一个输入；
3. 为1的Feature Map选择一个运算；
4. 为2的Feature Map选择一个运算；
5. 选择一个合并3，4得到的Feature Map的运算。

### 2、NASNet 搜索空间

15种操作：恒等连接，1x1（卷积），3x3（卷积、深度可分离卷积、空洞卷积、平均池化、最大池化）,1x3+3x1(卷积)，5x5（深度可分离卷积，最大池化），7x7（深度可分离卷积、最大池化），1x7+7x1(卷积)；

合并操作：（1）单位加；（2）拼接。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220507233855766.png width=75% />
</div>
### 3、NASNet 细节

1. 激活函数统一使用ReLU，实验结果表明ELU nonlinearity效果略优于ReLU；
2. 全部使用Valid卷积，padding值由卷积核大小决定；
3. Reduction Cell的Feature Map的数量需要乘以2，Normal Cell数量不变。初始数量人为设定，一般来说数量越多，计算越慢，效果越好；
4. Normal Cell的重复次数（N)人为设定；
5. 深度可分离卷积在深度卷积和单位卷积中间不使用BN或ReLU;
6. 使用深度可分离卷积时，该算法执行两次；
7. 所有卷积遵循ReLU->卷积->BN的计算顺序；
8. 为了保持Feature Map的数量的一致性，必要的时候添加1x1卷积。

### 4、NASNet 网络结构

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220507235912753.png width=75% />
</div>


（1）*Normal Cell*：输出Feature Map和输入Feature Map的尺寸相同；（2）*Reduction Cell*：输出Feature Map对输入Feature Map进行了一次降采样，在Reduction Cell中，对使用Input Feature作为输入的操作（卷积或者池化）会默认步长为2。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220507235957692.png width=75% />
</div>
### 5、Scheduled Drop Path 计划DropPath

在优化类似于Inception的多分支结构时，以一定概率随机丢弃掉部分分支是避免过拟合的一种非常有效的策略，例如DropPath。但是DropPath对NASNet不是非常有效。

在NASNet的Scheduled Drop Path中，丢弃的概率会随着训练时间的增加线性增加。这么做的动机很好理解：训练的次数越多，模型越容易过拟合，DropPath的避免过拟合的作用才能发挥的越有效。

## 实验结果

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220508000440227.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220508000344539.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220508000459903.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220508000514636.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220508000523898.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220508000536477.png width=75% />
</div>
## 参考文献

[^01]: [YaqiLYU-NAS之Google和CVPR2019相关论文-知乎](https://zhuanlan.zhihu.com/p/61149576)

[^02]: [大师兄-NASNet详解-知乎](https://zhuanlan.zhihu.com/p/52616166)

