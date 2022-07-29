# CBAM, Convolutional Block Attention Module

摘要：CBAM (Convolutional Block Attention Module)，可以在通道和空间维度上进行 Attention。
<!--more-->

## 文献信息
| 信息 | 内容                                                         |
| ---- | ------------------------------------------------------------ |
| 日期 | 2018.07                                                      |
| 作者 | Sanghyun Woo et al.                                          |
| 机构 | Korea Advanced Institute of Science and Technology           |
| 来源 | ECCV 2018                                                    |
| 链接 | [CBAM: Convolutional Block Attention Module](https://arxiv.org/abs/1807.06521) |
| 代码 |                                                              |

## 个人理解

><strong style="color:red;">问题:</strong> 注意力机制问题；
>
><strong style="color:red;">方法:</strong> 通道和空间结合的注意力机制；
>
><strong style="color:red;">结论:</strong> 在ImageNet-1K、MS COCO检测和VOC 2007检测数据集上的大量实验来验证CBAM；
>
><strong style="color:red;">理解:</strong> 
>
>1. 通道注意力机制：最大池化 + 平均池化（hw方向），共享衰减全连接 +ReLU, 共享衰减全连接 + sigmod，逐元素相乘；
>2. 空间注意力机制：最大池化 + 平均池化（c方向），cat + 7x7卷积（通道2变为1）+ sigmod，逐元素相乘；
>
><strong style="color:red;">优化：</strong>无。
---

## 背景知识

注意力不仅能告诉你该把注意力集中在哪里，还能提高兴趣的表现力。

作者的目标是通过使用注意机制来提高表征能力，关注重要特征并抑制不必要的特征。

## 原理方法

### 1、CBAM
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213925.png width=75% />
</div>


CBAM由2个独立的子模块：

- 通道注意力模块（Channel Attention Module，CAM) 
- 空间注意力模块（Spartial Attention Module，SAM) ，

不仅仅节约参数和计算力，且即插即用。

### 2、Channel Attention Module（CAM）与 Spatial Attention Module（SAM）
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213926.png width=75% />
</div>


CAM具体实现过程：

1. 将输入的特征图F（H×W×C）分别经过基于width和height的global max pooling（全局最大池化）和global average pooling（全局平均池化），得到两个1×1×C的特征图；
2. 再将它们分别送入一个两层的神经网络（MLP），第一层神经元个数为 C/r（r为减少率），激活函数为 Relu，第二层神经元个数为 C，这个两层的神经网络是共享的。
3. 将MLP输出的特征进行基于element-wise的加和操作；
4. 再经过sigmoid激活操作，生成最终的channel attention feature，即M\_c。
5. 最后将M\_c和输入特征图F做elemen-twise乘法操作，生成Spatial attention模块需要的输入特征。

针对池化方式，CBAM的通道注意力机制模块相对于SENet还引进了平均池化：作者发现**组合avg池化和max池化**的效果要更好，原因可能是最大池化丢失的信息太多，avg&max的组合连接方式比单一的池化丢失的信息更少，所以效果会更好一点。
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213926.png width=75% />
</div>


SAM具体实现过程：额外引进了空间的注意力机制。

1. 将Channel attention模块输出的特征图F‘作为本模块的输入特征图。
2. 先基于channel的global max pooling 和global average pooling，得到两个H×W×1 的特征图；
3. 再将2个特征图基于channel 做concat操作（通道拼接）。
4. 然后经过一个**7×7卷积（7×7比3×3效果要好）**操作，降维为1个channel，即H×W×1。
5. 再经过sigmoid，生成spatial attention feature，即M_s。
6. 最后将M\_s和输入特征图F做elemen-twise乘法操作，得到最终生成的特征。

针对卷积核大小，作者也发现平均池化核最大池化和7x7卷积更合适：
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213927.png width=75% />
</div>


### 3、CAM和SAM的组合形式

通道注意力和空间注意力这两个模块能够以并行或者串行顺序的方式组合在一块儿，关于通道和空间上的串行顺序和并行作者进行了实验对比，发现先通道再空间的结果会稍微好一点。
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213927.png width=75% />
</div>


CBAM和ResBlock组合：
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213928.png width=75% />
</div>


### 4、CBAM可视化

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213924.png width=75% />
</div>

## 实验结果

经典分类网络实验结果：
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213929.png width=75% />
</div>

轻量级网络实验结果：
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213929.png width=75% />
</div>


可视化实验结果：
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213930.png width=75% />
</div>


目标检测实验结果：
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213931.png width=75% />
</div>


## 参考文献

[^01]: [姚路遥遥-【注意力机制】CBAM详解-CSDN](https://blog.csdn.net/Roaddd/article/details/114646354)


