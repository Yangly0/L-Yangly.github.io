# MobileNets, Efficient Convolutional Neural Networks for MobileVision Applications

摘要：MobileNet V1是由google2016年提出，主要创新点在于深度可分离卷积。
<!--more-->

## 文献信息
| 信息 | 内容                                                         |
| ---- | ------------------------------------------------------------ |
| 日期 | 2017.04                                                      |
| 作者 | Andrew G. Howard et al.                                      |
| 机构 | Google Inc                                                   |
| 来源 | arXiv.                                                       |
| 链接 | [MobileNets: Efficient Convolutional Neural Networks for MobileVision Applications](https://arxiv.org/abs/1704.04861) |
| 代码 | [Code]()                                                     |

## 个人理解
><strong style="color:red;">问题:</strong> 从计算量角度考虑模型优化；
> 
><strong style="color:red;">方法:</strong> 深度可分离卷积+宽度和分辨率超参；
> 
><strong style="color:red;">结论:</strong> 在资源和精度权衡方面进行了大量实验，与其他流行的ImageNet分类模型相比，我们表现出了强大的性能。然后，展示了MobileNet在广泛的应用和用例中的有效性，包括目标检测、细粒度分类、人脸属性和大规模地理定位；
> 
><strong style="color:red;">理解:</strong> 
>1. 深度可分离卷积；
>2. 点卷积；
>3. 宽度和分辨率超参；
> 
><strong style="color:red;">优化：</strong>无。
---

## 背景知识

1. 为了追求分类准确度，模型深度越来越深，模型复杂度也越来越高。模型过于庞大，面临着内存不足的问题，其次这些场景要求低延迟，或者说响应速度要快。
2. 在某些真实的应用场景如移动或者嵌入式设备，如此大而复杂的模型是难以被应用的。
3. 一是对训练好的复杂模型进行压缩得到小模型；二是直接设计小模型并进行训练。不管如何，其目标在保持模型性能（accuracy）的前提下降低模型大小（parameters size），同时提升模型速度（speed, low latency）。

## 原理方法

### 1、模型复杂度与硬件性能的衡量

#### 模型复杂度的衡量

参数数量（Params）：指模型含有多少参数，直接决定模型的大小，也影响推断时对内存的占用量。
- 单位通常为 M，通常参数用 float32 表示，所以模型大小是参数数量的 4 倍左右
- 参数数量与模型大小转换示例：$10M  \ \  float32 \ \  bit = 10M \times 4 Byte = 40MB$

理论计算量（FLOPs）：指模型推断时需要多少计算次数。
- 是 floating point operations 的缩写（注意 s 小写），可以用来衡量算法/模型的复杂度，这关系到算法速度，大模型的单位通常为 G（GFLOPs：10亿次浮点运算），小模型单位通常为 M。
- 通常只考虑乘加操作(Multi-Adds)的数量，而且只考虑 CONV 和 FC 等参数层的计算量，忽略 BN 和 PReLU 等等。一般情况，CONV 和 FC 层也会 忽略仅纯加操作 的计算量，如 bias 偏置加和 shotcut 残差加等，目前有 BN 的卷积层可以不加 bias。
- PS：也有用 MAC（Memory Access Cost） 表示的。

#### 硬件性能的衡量
- 算力： 计算平台倾尽全力每秒钟所能完成的浮点运算数（计算速度），单位一般为 TFLOPS（floating point of per second）。
- 带宽： 计算平台倾尽全力每秒所能完成的内存（CPU 内存 or GPU 显存）交换量，单位一般为 GB/s（GByte/second），计算公式一般为 内存频率 × 内存位宽 / 8。

### 2、模型复杂度的计算公式

假设卷积核大小为$K_h \times K_w$，输入通道数为$C_{in}$，输出通道数为$C_{out}$，输出特征图的宽和高分别为W和H，忽略偏执项。

- Conv标准卷积层：
  - Parmas:$C_{in} \times K_h \times K_w \times C_{out}$
  - <div align=center>
        <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213851.png width=75% />
        <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213852.png width=75% />
        <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213852.png width=75% />
    </div>
- FC全连接层（相当于k=1）
  - Parmas:$C_{in} \times C_{out}$
  - FLOPs: $C_{in} \times C_{out}$  
- 参数量与计算量: `https://github.com/Lyken17/pytorch-OpCounter`。
- Group Conv：
  - Parmas:$(K^2 \times \frac{C_{in}}{g} \times \frac{C_{out}}{g}) \times g = k^2 C_{in} C_{out}/g$
  - FLOPs: $k^2 C_{in} C_{out}/g \times W \times H$
- Depthwise Conv (DWConv)：即特殊分组$g=C_{in}=C_{out}$
  - Parmas:$(K^2 \times \frac{C_{in}}{g} \times \frac{C_{out}}{g}) \times g = k^2 C_{in}$
  - FLOPs: $k^2 C_{in} \times W \times H$

### 3、MobileNet

1. Depthwise separable convolution

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213853.png width=50% />
	<img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213854.png width=50% />
</div>


深度级可分离卷积其实是一种可分解卷积操作（factorized convolutions），其可以分解为两个更小的操作：depthwise convolution和pointwise convolution，分别起到滤波和线性组合的作用，同时减少参数量和计算量。

- **TODO:如何分离的？**

Depthwise convolution和标准卷积不同，对于标准卷积其卷积核是用在所有的输入通道上（input channels），而depthwise convolution针对每个输入通道采用不同的卷积核，就是说一个卷积核对应一个输入通道，所以说depthwise convolution是depth级别的操作。

而pointwise convolution其实就是普通的卷积，只不过其采用1x1的卷积核。

参数量和计算量：

$$
\#params=k^2 c_{in} + c_{in}c_{out}  \\
\#MultiAdd=k^2 c_{in} \times h_{out} w_{out} + 1 \times 1 \times c_{in}c_{out} \times h_{out} w_{out}
$$
相比于标准卷积，理论上的加速比例可达：
$$
\frac{k^2 c_{in} \times h_{out} w_{out} + 1 \times 1 \times c_{in}c_{out} \times h_{out} w_{out}}{k^2 c_{in} c_{out} \times h_{out} w_{out}} = \frac{1}{c_{out}} + \frac{1}{k^2}
$$
若k=3，参数量大约会减少到原来的 1/8 → 1/9。

Note：原论文中对第一层没有用此卷积，深度可分离卷积中的每一个后面都跟 BN 和 RELU。

Note：采用 depth-wise convolution 会有一个问题，就是导致信息流通不畅，即输出的 feature map 仅包含输入的 feature map 的一部分，而MobileNet 采用了 point-wise(1*1) convolution 帮助信息在通道之间流通。

2. Pooling layer

Global Average Pooling：这一层没有参数，计算量可以忽略不计。

CONV/s2（步进2的卷积）代替 MaxPool+CONV：使得参数数量不变，计算量变为原来的 1/4 左右，且省去了MaxPool 的计算量。

3. 两个超参数Width Multiplier和Resolution Multiplier

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213854.png width=75% />
</div>


- Width Multiplier($\alpha$): Thinner Models， 目的是使模型变瘦。
    - 所有层的通道数（channel） 乘以$\alpha$ 参数(四舍五入)，模型大小近似下降到原来的$\alpha^{2}$倍，计算量下降到原来的 $\alpha^{2}$倍
    - $\alpha \in (0, 1]$ with typical settings of 1, 0.75, 0.5 and 0.25，降低模型的宽度。
- Resolution Multiplier($\rho$): Reduced Representation，目的是降低图片的分辨率。
    - 输入层的分辨率（resolution）乘以$\rho$参数(四舍五入)，等价于所有层的分辨率乘$\rho$，模型大小不变，计算量下降到原来的 $\rho^{2}$倍。
    - $rho \in (0, 1]$降低输入图像的分辨率，一般输入图片的分辨率是224, 192, 160 or 128。

计算量：
$$
\#MultiAdd=k^2 \times \alpha c_{in} \times \alpha c_{out} \times \rho h_{out} \times \rho w_{out} \\ + 1 \times 1 \times \alpha c_{in} \times \alpha c_{out} \times \rho h_{out} \times \rho w_{out}
$$

### 4、网络结构

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213855.png width=75% />
</div>

## 实验结果

实验结果如下，MobileNet采用k=3的卷积核，所以一般可达8-9倍加速，而精度不损失太多。更多有意思的细节和实验请参考原文。关于超参数的选择，准确度和参数量和参数运算量的关系，之间有个trade off，合理选择参数即可。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213856.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213856.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213857.png width=75% />
    <imghttps://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220429203829899.png src= width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213858.png width=75% />
</div>
## 参考文献
[^01]: [kai.han-轻量级CNN之MobileNet系列-知乎](https://zhuanlan.zhihu.com/p/45209964)


