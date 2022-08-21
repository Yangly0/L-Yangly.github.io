# Searching for MobileNetV3

摘要：文章的亮点在于**网络的设计利用了NAS（network architecture search）算法以及NetAdapt algorithm算法**。
<!--more-->

# Searching for MobileNetV3[^01][^02]

## 文献信息

| 信息 | 内容                                                         |
| ---- | ------------------------------------------------------------ |
| 日期 | 2019.05                                                      |
| 作者 | Andrew Howard et al.                                         |
| 机构 | Google AI, Google Brain                                      |
| 来源 | ICCV 2019                                                    |
| 链接 | [Searching for MobileNetV3](https://arxiv.org/abs/1905.02244) |
| 代码 | [Code]()                                                     |

## 个人理解

><strong style="color:red;">问题:</strong> 文章为了解决轻量化网络；
> 
><strong style="color:red;">方法:</strong> 文章提出了NAS+Netadapt，SE，hswish，细节调整；
> 
><strong style="color:red;">结论:</strong> MobileNetV3 Large在ImageNet分类上的准确率比MobileNetV2高3.2%，同时延迟降低了20%。与延迟相当的MobileNetV2模型相比，MobileNetV3 Small的准确率高出6.6%。MobileNetV3大检测速度快25%以上，与MobileNetV2在COCO检测上的准确度大致相同。MobileNetV3大型LRASPP比MobileNetV2 R-ASPP快34%，在城市景观分割方面具有相似的精度；
> 
><strong style="color:red;">理解:</strong> 轻量化网络，通过网络搜索寻找最优网络，并引进注意力机制和新的激活函数。
> 
><strong style="color:red;">优化：</strong>无。
---

## 背景知识

轻量化模型：
1. 基于轻量化网络设计：比如mobilenet系列，shufflenet系列， Xception等，使用Group卷积、1x1卷积等技术减少网络计算量的同时，尽可能的保证网络的精度。
2. 模型剪枝： 大网络往往存在一定的冗余，通过剪去冗余部分，减少网络计算量。
3. 量化：利用TensorRT量化，一般在GPU上可以提速几倍。
4. 知识蒸馏：利用大模型（teacher model）来帮助小模型（student model）学习，提高student model的精度。

## 原理方法

### 1、NAS

**资源受限的NAS**，用于在计算和参数量受限的前提下搜索网络来优化各个块（block），所以称之为**模块级搜索（Block-wise Search**。

网络架构搜索(NAS)源于18年谷歌的一篇文章 “MnasNet: Platform-Aware Neural Architecture Search for Mobile”，其优势是相比于传统手动调参确定架构，可以搜索到准确率和效率之间权衡。

### 2、NetAdapt

**NetAdapt**，用于对各个模块确定之后网络层的微调每一层的卷积核数量，所以称之为**层级搜索（Layer-wise Search）**。

### 3、架构上的调整

1. MobileNet v1和v2都从具有32个滤波器的常规3×3卷积层开始，然而实验表明，这是一个相对耗时的层，只要16个滤波器就足够完成对224 x 224特征图的滤波。虽然这样并没有节省很多参数，但确实可以提高速度。

![image-20220506130839211](C:/Users/ly/AppData/Roaming/Typora/typora-user-images/image-20220506130839211.png)

2. 在MobileNetV2中，在全局平均池化层之前，是一个1×1卷积，将通道数从320扩展到1280，因此就能得到更高维度的特征，供分类器层使用。 这样做的好处是，在预测时有更多更丰富的特征来满足预测，但是同时也**引入了额外的计算成本与延时**。所以，需要改进的地方就是要**保留高维特征的前提下减小延时**。

   在MobileNetV3中，这个1 x 1卷积层位于全局平均池化层的后面，因此它可用于更小的特征图，因此速度更快。如图所示， 这样使我们就能够删除前面的bottleneck层和depthwise convolution层，而不会降低准确性。

   可以看到，现在最后两个卷积后面都没有BN层了。由于这些改进，虽然MobileNetV3使用了与MnasNet相同类型的构造块，而且具有更多参数，MobileNet v3仍比MnasNet更快，并且具有相同的精度。

### 4、激活函数h-swish

![image-20220506130951844](C:/Users/ly/AppData/Roaming/Typora/typora-user-images/image-20220506130951844.png)

谷歌发现新的[swish](https://arxiv.org/abs/1710.05941)激活函数比relu和sigmoid显著提高了精度，但是由于sigmoid在移动设备中很难计算，所以选择用分段线性近似来拟合sigmoid，
$$
sigmod = \frac{ReLU6(x+3)}{6}
$$


进一步，swish变为“硬件友好”的h-swish,

$$
h-swish= x \frac{ReLU6(x+3)}{6}
$$

> 不过，并非整个模型都使用了h-swish，模型的前一半层使用常规ReLU（**第一个conv层之后的除外**）。 为什么要这样做呢？因为作者发现，h-swish仅	在更深层次上有用。 此外，考虑到特征图在较浅的层中往往更大，因此计算其激活成本更高，所以作者选择在这些层上简单地使用ReLU（**而非ReLU6**），因为它比h-swish省时。
>
> **具体解释一下如何完成ReLU6(x + 3) / 6的。如图2所示，在Mul层中，做了乘以0.16667的乘法，这就相当于除以6；ReLU6则融合在了卷积层之中；另外，对于x+3，这里的3被加在了卷积层的偏置层中了。这样做也是一种小的优化方式。**
>
> Swish 具备不饱和，光滑，非单调性，
> $$
> swish = x * sigmod(x)  \\ 
> swish = x * sigmod(\beta x)  \\ 
> $$
>
> 1. 饱和激活函数（$lim_{n->\infty}h'(x) = 0$）会导致神经网络的性能大幅度下降，从而产生梯度消失问题,如常见的sigmod或tanh函数都存在该问题;
> 2. 平滑和一阶导数,二阶导数平滑的特性;
> 3. 非单调性，非单调性确与其他常见的激活函数不同；
>
> 谷歌测试证明，Swich适应于局部响应归一化，并且在40以上全连接层的效果要远优于其他激活函数，而在40全连接层之内则性能差距不明显。但是根据在mnist数据上AleNet的测试效果却证明，Swich在低全连接层上与Relu的性能差距依旧有较大的优势。

### 5、SE结构

![image-20220506132753589](C:/Users/ly/AppData/Roaming/Typora/typora-user-images/image-20220506132753589.png)

在bottlenet结构中加入了SE结构，并且放在了depthwise filter之后，但是SE结构会消耗一定的时间，所以作者在含有SE的结构中，将expansion layer的channel变为原来的1/4，这样即提高了精度，同时还没有增加时间消耗，并且SE结构放在了depthwise之后。

### 6、V3网络结构

![image-20220506133953222](C:/Users/ly/AppData/Roaming/Typora/typora-user-images/image-20220506133953222.png)

![image-20220506134004601](C:/Users/ly/AppData/Roaming/Typora/typora-user-images/image-20220506134004601.png)

MobileNet v3仍然使用宽度乘子的概念。 因此，有很多方法可以调整模型的大小和形状。 与之前的版本一样，该架构不仅定义了单个模型，还定义了一系列的变体：

- Large：就如前所述的
- Small：包含了更少的构造块和更少的滤波器
- Large minimalistic：和Large类似，但是没有SE、h-swish和5 x 5卷积
- Small minimalistic：和Small的变体类似

### 7、Lite Reduced Atrous Spatial Pyramid Pooling (LR-ASPP)

![image-20220506170533358](C:/Users/ly/AppData/Roaming/Typora/typora-user-images/image-20220506170533358.png)

Lite R-ASPP是对R-ASPP的改进，它以一种类似于挤压和激励模块的方式部署了全局平均池，在该模块中，使用了一个大的池化核和大步长（以节省一些计算），再结合1x1卷积，sigmod并上采样对结果加权，同时还引进了1/8特征图信息。对MobileNetV3的最后一块应用Atrus卷积，以提取更密集的特征，并进一步从低级特征中添加跳过连接，以捕获更详细的信息。

## 实验结果

![image-20220506134102284](C:/Users/ly/AppData/Roaming/Typora/typora-user-images/image-20220506134102284.png)

![image-20220506134140860](C:/Users/ly/AppData/Roaming/Typora/typora-user-images/image-20220506134140860.png)

![image-20220506134235778](C:/Users/ly/AppData/Roaming/Typora/typora-user-images/image-20220506134235778.png)

## 参考文献
[^01]: [春枫琰玉-mobilenet系列之又一新成员---mobilenet-v3-CSDN](https://blog.csdn.net/Chunfengyanyulove/article/details/91358187)

[^02]: [SayStayy-理解MobileNetV3-知乎](https://zhuanlan.zhihu.com/p/260790699)


