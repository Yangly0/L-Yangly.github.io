# Rethinking Bottleneck Structure for Efficient Mobile Network Design

摘要：依图科技&新加坡国立大学颜水成团队提出的一种对标MobileNetV2的网络架构MobileNeXt。它针对MobileNetV2的核心模块逆残差模块存在的问题进行了深度分析，提出了一种新颖的SandGlass模块，并用于组建了该文的MobileNeXt架构，SandGlass是一种通用的模块，它可以轻易的嵌入到现有网络架构中并提升模型性能。
<!--more-->

## 文献信息
| 信息 | 内容                                                         |
| ---- | ------------------------------------------------------------ |
| 日期 | 2020.07                                                      |
| 作者 | Zhou Daquan et al.                                           |
| 机构 | National University of Singapore, Yitu Technology            |
| 来源 | ECCV                                                         |
| 链接 | [Rethinking Bottleneck Structure for Efficient Mobile Network Design](https://arxiv.org/abs/2007.02269) |
| 代码 | [zhoudaquan/rethinking_bottleneck_design](https://github.com/zhoudaquan/rethinking_bottleneck_design) |

## 个人理解

><strong style="color:red;">问题:</strong> 优化MobileNet v2 的逆残差模块；
>
><strong style="color:red;">方法:</strong> 沙漏模块；
>
><strong style="color:red;">结论:</strong> 在ImageNet分类任务中，通过简单的模块替换(即采用SandGlass 替换MobileNetV2中的瓶颈残差块)，即可取得了1.7%的性能提升，且不会导致额外的参数量与计算量提升；在VOC2007测试集上，按到目标检测指标的0.9%mAP的提升。作者将所提模块嵌入到NAS方法(DARTS)搜索空间中，取得了0.13%的性能提升且参数量降低25%；
>
><strong style="color:red;">理解:</strong> 两个depthwise卷积和两个pointwise卷积组成，部分卷积不需激活以及shorcut建立在高维度特征上；
>
><strong style="color:red;">优化：</strong>无；
---

## 背景知识

逆残差模块已成为手机端网络架构设计的主流架构。它通过引入两个主要的设计规则(1.逆残差学习；2.线性瓶颈层)对经典的残差瓶颈模块进行了改变。但是这种设计模块可能导致信息损失与梯度混淆。

## 原理方法

### 1、瓶颈块与逆残差块

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213938.png width=75% />
</div>


- Bottleneck：它包含两个1x1卷积(分别进行降维与升维)与一个3x3卷积(用于空间信息变换)，它是一种heavy-weight模块；
- Inverted Residual Block：它包含两个1x1卷积(分别进行升维与降维)与一个3x3深度可分离卷积(用于空间信息变换)，它是一种light-weight模块。

### 2、SandGlass 沙漏模块

(1) 更宽的网络有利于缓解梯度混淆问题，并有助于提升模型性能；(2)逆残差模块中的短连接可能会影响梯度回传。

考虑到上述逆残差模块的局限性，作者对其设计规则进行重思考并提出了SandGlass模块缓解上述问题。该模块的设计主要源自如下几点分析：

- 保持更多的信息从bottom传递给top层，进而有助于梯度回传；
- 深度卷积是一种轻量型单元，可以执行两次深度卷积以编码更多的空间信息。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213938.png width=75% />
</div>



- Position of Expansion and Reduction. 在原始的逆残差模块中先进行升维再进行降维。基于前述分析，为确保高维度特征的短连接，作者对两个1x1卷积的顺序进行了调整。假设$F \in R^{D_f \times D_f \times M}$表示输入张量，$G \in R^{D_f \times D_f \times M}$表示输出张量(注：此时尚未考虑深度卷积)，那么该模块的可以写成如下形式，中间两个1x1卷积，$\phi_e$和$\phi_r$用于维度扩展和缩减的pointwise卷积。这样的设计将bottleneck保持在residual path中间能够减少参数量和计算量，最重要的是，能将shortcut建立在维度较大的特征上。
$$
G = \phi_e(\phi_r(F)) + F
$$

- High-dimensional Shortcut. 作者并未在瓶颈层间构建Shortcut，而是在更高维特征之间构建Shortcut。更宽的短连接有助于更多的信息从输入F传递给输出G，从而更好地传递信息和回传梯度。
- Learning expressive spatial features. 1x1 pointwise 卷积有助于编码通道间的信息，但难以获取空间信息，因此，在这里作者沿着逆残差模块的思路引入深度卷积编码空间信息。不同于逆残差模块在两个1x1卷积之间引入深度卷积，作者认为1x1卷积导致了减少的空域信息编码，因此将深度卷积置于两个1x1卷积之外，见图中的两个3x3深度卷积。该模块可以采用如下公式进行描述：

$$
\hat{G} = \phi_{1,p} \phi_{1,d} (F) \\
G = \phi_{1,d} \phi_{2,p} (\hat(G)) + F 
$$

 其中$\phi_{1,p}, \phi_{1,d}$分别表示1x1卷积与深度卷积。从而确保了深度卷积在高维空间处理并得到更丰富的特征表达。

- Activation Layer. 已有研究表明：线性瓶颈层有助于避免特征出现零化现象(防止特征值变为零)，进而导致信息损失。基于此，作者在用于降维的1x1卷积后不添加激活函数。同时最后一个深度卷积后也不添加激活函数，激活函数仅加第一个深度卷积与最后一个1x1卷积之后。
- Block Structure. 基于上述考虑，得到了该文所设计的新颖的残差瓶颈模块。但当输入与输出通道数不相同时不进行短连接操作。depthwise卷积采用 $3 \times 3 $ 卷积核，在需要的地方采用BN+ReLU6的组合。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213939.png width=75% />
</div>

### 3、MobileNeXt 结构

作者将上述模块构建的网络架构称之为MobileNeXt。注：SandGlass中的扩展比例与MobileNetV2中的相同。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213940.png width=75% />
</div>


### 4、Identity tensor multiplier 恒等张量乘子

**残差模块中的短连接有助于梯度跨层传播。**但是，作者通过实验发现：没有必要保持全局恒等tensor与残差分支组合。为使得该网络更适合于手机端，作者引入了一个新的超参数：identity tensor multiplier，表示为$\alpha \in [0, 1]$。为简单起见，假设$\phi$表示残差分支的变换函数，那么添加该超参数后的模块可以重写为：
$$
G_{1: \alpha M}=\phi(F)_{1: \alpha M}+F_{1: \alpha M} \\
G_{\alpha M: m}=\phi(F)_{\alpha M: M}
$$

引入的超参数$\alpha$有两个作用：

- 降低该超参数，每个模块中的add数量可以进一步降低，因为add操作会占用不少耗时。用户可以选择更少的$\alpha$以得到更好的推理速度且性能几乎无影响；
- 降低内存访问时间。影响模型推理的一个重要因素是，内存访问消耗(Memory acces cost, MAC)。降低该超参数有助于减少cache占用，进而加速推理。

## 实验结果

为说明所提方案的有效性，作者在ImageNet与VOC数据集上进行了实验分析。在模型训练过程中，优化器为SGD(momentum=0.9,weight_decay=$4\times 10^{-5}$)，初始学习率为0.05，cosine方式衰减，BatchSize=256，4个GPU。如无特殊说明，模型总计训练200epoch。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213940.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213941.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213942.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213943.png width=75% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com//images/20220706213943.png width=75% />
</div>

## 参考文献

[^01]: [风度78-【论文解读】打破常规，逆残差模块超强改进，新一代移动端模型MobileNeXt来了！精度速度双超MobileNetV2...-CSDN](https://blog.csdn.net/fengdu78/article/details/107241361)


