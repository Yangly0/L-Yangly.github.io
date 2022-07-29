# Very Deep Convolutional Networks for Large-Scale Image Recognition

摘要：VGG 小卷积替换大卷积，减少网络参数，提高网络深度。
<!--more-->

## 文献信息

| 信息 | 内容                                                         |
| ---- | ------------------------------------------------------------ |
| 日期 | 2014.09                                                      |
| 作者 | Karen Simonyan et al.                                        |
| 机构 | Oxford, [Visual Geometry Group Home Page](http://www.robots.ox.ac.uk/~vgg/research/very_deep/) |
| 来源 | ICLR 2015                                                    |
| 链接 | [Very Deep Convolutional Networks for Large-Scale Image Recognition](https://arxiv.org/abs/1409.1556) |
| 代码 | [Code]()                                                     |

## 个人理解
><strong style="color:red;">问题:</strong> 网络深度和卷积核大小探讨；
>
><strong style="color:red;">方法:</strong> 采用连续的几个3x3的卷积核代替AlexNet中的较大卷积核（11x11，7x7，5x5）；
>
><strong style="color:red;">结论:</strong> ImageNet ILSVRC-2014 定位第一名 分类第二名；
>
><strong style="color:red;">理解:</strong> 
>
>1. 3x3卷积堆叠替换大卷积核，保证感受野不变的同时，增加了网络深度(非线性能力)，同时减少参数（卷积核大小）。
>2. 引入1x1的卷积核，在不影响输入输出维度的情况下，引入非线性变换，增加网络的表达能力，降低计算量。
>
><strong style="color:red;">优化：</strong> 
>
>1. 后端采用了3个全连接，耗费计算资源，可以用全局平均池化替换。
>2. BN操作。
---

## 背景知识

## 原理方法

VGGNet是VGG是牛津大学 Visual Geometry Group（视觉几何组）的缩写，以研究机构命名。。

VGG主要探究了卷积神经网络的深度和其性能之间的关系，相比AlexNet的改进是采用连续的几个3x3的卷积核代替AlexNet中的较大卷积核（11x11，7x7，5x5）。

### 1、卷积替换

**核心思想**：以3x3卷积替换5x5卷积为例，5x5卷积看做一个小的全连接网络在5x5区域滑动，可以先用一个3x3的卷积滤波器卷积，然后再用一个全连接层连接这个3x3卷积输出，这个全连接层也可以看做一个3x3卷积层。这样就可以用两个3x3卷积级联（叠加）起来代替一个 5x5卷积。[^01]

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213815.png width=75% />
</div>



### 2、VGG网络结构

网络参数：**VGG-16的16指的是conv+fc的总层数是16，是不包括max pool的层数**。

![image-20220426103251065](C:/Users/ly/AppData/Roaming/Typora/typora-user-images/image-20220426103251065.png)

### 3、预训练

为了解决初始化（权重初始化）等问题，VGG采用的是一种预训练方式，先训练浅层的的简单网络 VGG11，再复用 VGG11 的权重来初始化 VGG13，如此反复训练并初始化VGG19，能够使训练时收敛的速度更快。

## 训练测试

1. 训练设置：批量大小设为 256，动量为 0.9。训练通过权重衰减（L2 惩罚乘子设定为 5×10^−4）进行正则化，前两个全连接层采取 dropout 正则化（dropout 比率设定为 0.5）。学习率初始设定为 10−2，然后当验证集准确率停止改善时，学习率以 10 倍的比率进行减小。学习率总共降低 3 次，学习在 37 万次迭代后停止（74 个 epochs）。
2. 数据增强：裁剪图像、随机水平翻转和随机RGB颜色偏移。
3. 训练时，先训练级别简单（层数较浅）的VGGNet的A级网络，然后使用A网络的权重来初始化后面的复杂模型，加快训练的收敛速度。
4. 采用Multi-Scale的方法来训练和预测。可以增加训练的数据量，防止模型过拟合，提升预测准确率。

## 参考文献
[^01]:[Amusi-一文读懂VGG网络-知乎](https://zhuanlan.zhihu.com/p/41423739)


