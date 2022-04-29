# Going deeper with convolutions

摘要：针对网络越来越大的问题，GooLeNet提出了将全连接的结构变为稀疏的全连接结构。然而，运算设备在处理不均匀的稀疏数据时运算效率很低。GoogLeNet将稀疏矩阵聚合成稠密矩阵，构建Inception结构解决稀疏运算效率问题。
<!--more-->

# Going deeper with convolutions

## 文献信息
| 信息 | 内容                                                         |
| ---- | ------------------------------------------------------------ |
| 日期 | 2014.09                                                      |
| 作者 | Christian Szegedy et al.                                     |
| 机构 | Google Inc                                                   |
| 来源 | arXiv                                                             |
| 链接 | [Going deeper with convolutions](https://arxiv.org/abs/1409.4842) |
| 代码 | [Code]()                                                     |

## 个人理解
><strong style="color:red;">问题:</strong> 提升网络性能的手段有两个：增加网络的深度和宽度，但是这会带来两个明显的问题，
>
>1. 参数太多，如果训练数据集有限，很容易产生过拟合；
>
>2. 网络越大、参数越多，计算复杂度越大，难以应用；
>
>3. 网络越深，容易出现梯度弥散问题（梯度越往后穿越容易消失），难以优化模型。
>
><strong style="color:red;">方法:</strong> 为了解决这些问题，文章提出了将全连接的结构变为稀疏的全连接结构。然而，运算设备在处理不均匀的稀疏数据时运算效率很低。GoogLeNet将稀疏矩阵聚合成稠密矩阵，构建Inception结构解决稀疏运算效率问题。
>
><strong style="color:red;">结论:</strong> ILSVRC 2014冠军；
>
><strong style="color:red;">理解:</strong> 
>
>1. Inception结构:自主搜索最优结构；
>2. Inception结构:不同尺度卷积核共同作用，提取不同感受野的特征，效果更好。
>3. 稠密结构：1×1和3×3卷积核看成是5×5卷积核的稀疏表示，相当于填充0为5x5卷积核，构成稀疏矩阵。
>
><strong style="color:red;">优化：</strong>略。
---

## 背景知识
略。
## 原理方法

### 1、原理
Inception网络的核心思想是找到卷积视觉网络可以近似的最优局部稀疏结构，并且该结构可以利用现有的密度矩阵计算硬件实现。

### 2、Inception 结构

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427095558750.png width=75% />
</div>

结构（a）：1x1, 3x3, 5x5 , 3x3 max pooling

1. 借鉴Serre 等人使用了一系列不同尺寸的固定的 Gabor 滤波器来解决多尺度的问题。
2. 搜索最优的稀疏结构。
3. 采用大小不同的卷积核，意味着感受野的大小不同，就可以得到不同尺度的特征。
4. 网络结构与稀疏，稠密关系：实际上可以将 1 × 1和 3 × 3 看成是 5 × 5 的稀疏表示，只不过是一种特殊的稀疏表示。因为如果是一般形式的稀疏表示的话，那么在 5×5 的矩阵中，哪个位置是 0 都是可以的，但是 1 × 1 和 3 × 3 却只有中心位置的数值是非零的，其余部分的数值都是0。这也说明了为什么作者认为这是一种 “使用密集组建的近似和覆盖”。

结构（b）：结构（a）+ 1X1的卷积核

1. Network-in-Network 是Lin等人提出的，网络中增加了额外的1×1的卷积，用以增加网络的深度，用于增强网络表示能力。
2. 5x5卷积核计算量大，1x1卷积核先降低维度再卷积，减少计算瓶颈。
3. 增加网络层数，加入非线性，提高网络的表达能力。因为一层可能会有多个卷积核，在同一个位置但在不同通道的卷积核输出结果相关性极高。一个1×1的卷积核可以很自然的把这些相关性很高，在同一个空间位置，但不同通道的特征结合起来。而其它尺寸的卷积核（3×3，5×5）可以保证特征的多样性。

### 1、GoogLeNet 网络

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427095703843.png width=75% />
</div>

1. 模块化的结构（Inception结构）：方便增添和修改,但保持低层为传统卷积方式不变，只在较高的层开始用Inception模块。由于技术的原因（训练时的内存利用率），这并不是完全必要的，只是反映了当前实现中的一些基础设施效率低下。
2. 平均池化：替换全连接层，源于NIN（Network in Network），事实证明这样可以将准确率提高0.6%。
3. Dropout：移除全连接，但仍然使用Dropout。
4. 辅助分类器：2个辅助的softmax用于向前传导梯度，解决梯度消失，加速网络收敛。

## 训练测试

### 1、训练参数

1. 输入图像（归一化）：224*224的RGB，减去均值。
2. 数据增强：图像8%到100%，长宽比在3/4到4/3之间，光度扭曲减轻过拟合，随机插值 。
3. 损失函数：Softmax，网络末尾的Softmax加上两个辅助分类器的Softmax x 0.3。  
4. 辅助分类器结构：5x5 ave-pooling（3 stride）+ 1x1卷积层（输出128） + 全连接层（1024） + ReLU + Softmax。  
5. 训练参数：动量0.9异步随机梯度下降，学习率每8轮下降4%，用Polyak Averaging创造最后的模型。  

### 2、实验结果

分类任务：ILSVRC2014

1. 训练7个模型，初始化方式相同，学习率下降方式相同，训练样本不同。
2. 数据增强（多尺度resize + 裁剪 + 翻转）：将图片短边resize到256，288，320，352，从中取左中右三个正方形。裁剪四角+中心的224x224，翻转，正方形resize到224x224，翻转。这样每张图片可以取得4x3x6x2的样本。  
3. 对softmax的结果做平均输出。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427101309552.png width=75% />
</div>



检测任务：

2015年，Girshick 等人提出了目标检测的 SOTA算法R-CNN。R-CNN将整个检测问题分解为两个子问题：利用颜色和纹理等低级线索以类别不可知的方式生成对象位置建议，并使用CNN分类器在这些位置识别对象类别。这种两阶段的方法利用了低水平线索边界框分割的准确性，以及先进CNN的强大分类能力。论文对两个阶段都进行了改进，例如针对更高对象边界框调用的多框预测，以及用于更好地分类边界框建议的集成方法。1.作为区域分类，随着初始模型的增加而增加；2.通过将选择性搜索方法与多框预测相结合，改进了区域建议步骤，以实现更高的对象边界框调用。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427101340421.png width=75% />
</div>

## 参考文献

[^01]: [任乾-深度学习|经典网络：GoogLeNet（一）-知乎](https://zhuanlan.zhihu.com/p/73857137)

