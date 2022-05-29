# Identity Mappings in Deep Residual Networks

摘要：ResNet v2 恒等连接的支路详细讨论与实验。
<!--more-->



# Identity Mappings in Deep Residual Networks[^01]

## 文献信息
| 信息 | 内容                                                         |
| ---- | ------------------------------------------------------------ |
| 日期 | 2016.03                                                      |
| 作者 | [Kaiming He](http://kaiminghe.com/) et al.                   |
| 机构 | Microsoft Research                                           |
| 来源 | ECCV2016                                                     |
| 链接 | [Identity Mappings in Deep Residual Networks](https://arxiv.org/abs/1603.05027) |
| 代码 | [KaimingHe](https://github.com/KaimingHe)/[resnet-1k-layers](https://github.com/KaimingHe/resnet-1k-layers) |

## 个人理解

>
><strong style="color:red;">问题:</strong> 残差块结构分析讨论，是否影响网络收敛速度和泛化能力。；
>
><strong style="color:red;">方法:</strong> 分析残差网络基本构件（`residual building block`）中的信号传播，本文发现当使用恒等映射（identity mapping）作为快捷连接（skip connection）并且将激活函数移至加法操作后面时，前向-反向信号都可以在两个 block 之间直接传播而不受到任何变换操作的影响。**同时大量实验结果证明了恒等映射的重要性**。
>
><strong style="color:red;">结论:</strong> 网络更易于训练并且泛化性能提升；
>
><strong style="color:red;">理解:</strong> 
>
>- 证明恒等映射（`identity mapping`）作为快捷连接（`skip connection`）对于残差块的重要性。
>- 残差单元将激活函数（先BN再ReLU）移到权值层之前，即**预激活（pre-activation）**，而**后激活（post-activation）**，并且预激活的单元中的所有权值层的输入都是归一化的信号，使得网络更易于训练并且泛化性能也得到提升。
>- 注意：预激活模式可能仅仅适用于深层模型，如论文中实验的ResNet-110。
>
><strong style="color:red;">优化：</strong>还有什么值得改进与优化的。
---

## 背景知识

## 原理方法

### 1、残差单元

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/v2-e3f649b10152fba79782fa0f511c1c6a_r.jpg width=75% />
</div>

残差单元公式:
$$
y_l=h(x_l)+F(x_l,W_l)
$$
$$
								x_{l+1} = f(y_l)
$$
其中，$x_l$是第l个残差单元的输入特征，$W=\{W_{l,k|1≤k≤K}\}$是一个与第 l个残差单元相关的权重和偏差的集合，$K$是残差单元内部的层的数量（$K$分别为2和3，即普通残差块和瓶颈残差块）。 $F$是残差函数，函数$f$是元素加和后的激活操作（ResNet-v1中采用的是ReLU）。

**默认$h(x_l)=x_l$，**假设$f$为**恒等映射**，$x_{l+1}\equiv x_{l}$，得：
$$
x_{l+1}=x_{l}+\mathcal{F}(x_{l},\mathcal{W}_{l}) \text{ }\text{ }\text{ }\text{ }\text{ }\text{ }\text{ }\text{ }\text{ }
$$


递归地，$x_{l+2}=x_{l+1}+\mathcal{F}(x_{l+1},\mathcal{W}_{l+1})=x_{l}+\mathcal{F}(x_{l},\mathcal{W}_{l})+\mathcal{F}(x_{l+1},\mathcal{W}_{l+1})$,得：

$$
x_{L}=x_{l}+\sum_{i=1}^{L-1}\mathcal{F}(x_{i},\mathcal{W}_{i}) \text{ }\text{ }\text{ }\text{ }\text{ }\text{ }\text{ }\text{ }\text{ }
$$


特性：
1. 任意深层单元的特征都可以由起始特征$x0$与先前所有残差函数相加得到，而普通通网络的深层特征是由一系列的矩阵向量相乘得到。**残差网络是连加$x_{L}=x_{0}+\sum_{i=0}^{L-1}\mathcal{F}(x_{i},\mathcal{W}_{i})$，普通网络是连乘**$\prod_{i=0}^{L-1}W_{i}{x}_0$（忽略ReLU和BN）。
2. 反向传播特性，假设损失函数为E，**残差结构表明梯度$\frac{\partial E}{\partial {{x}_{l}}}$可以被分解成两个部分进行反向传播**：其中 $\frac{\partial E}{\partial {{x}_{L}}}$直接传递信息而不涉及任何权重层，而另一部分$\frac{\partial E}{\partial {{x}_{L}}}\left(\frac{\partial {\sum_{i=l}^{L-1}\mathcal{F}}}{\partial {{x}_{l}}}\right)$表示通过权重层传播的信息。$\frac{\partial E}{\partial {{x}_{L}}}$保证了信息能够直接传回任意浅单元l。等式同样表明了在一个mini-batch中梯度$\frac{\partial E}{\partial {{x}_{l}}}$不可能出现消失的情况，因为通常$\frac{\partial {\sum_{i=l}^{L-1}\mathcal{F}}}{\partial {{x}_{l}}}$对于一个mini-batch的全部样本不可能都为-1。这意味着，哪怕权重是任意小的，也不可能出现梯度消失的情况。

$$
\frac{\partial E}{\partial {{x}_{l}}}=\frac{\partial E}{\partial {{x}_{L}}}\frac{\partial {{x}_{L}}}{\partial {{x}_{l}}}=\frac{\partial E}{\partial{{x}_{L}}}\left(1+\frac{\partial {\sum_{i=l}^{L-1}\mathcal{F}({x}_{i}, \mathcal{W}_{i})}}{\partial {{x}_{l}}}\right) \text{ }\text{ }\text{ }\text{ }\text{ }\text{ }\text{ }\text{ }\text{ }
$$

### 2、恒等跳跃连接

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/v2-a90049141bc4cc28ff871f920bae8124_r.jpg width=75% />
</div>

验证公式$h$是一个恒等连接的重要性，假设$h(x_{l})=\lambda_{l} x_{l}$，$λ_l$ 是可调节的变量，假设 $f$是恒等映射，得：
$$
{x}_{L} =  (\prod_{i=l}^{L-1}\lambda_{i}){x}_{l} + \sum_{i=l}^{L-1} (\prod_{j=i+1}^{L-1}\lambda_{\tiny j}) \mathcal{F}({x}_{i}, \mathcal{W}_{i})  \\ 

{x}_{L} = (\prod_{i=l}^{L-1}\lambda_{i}){x}_{l} + \sum_{i=l}^{L-1}\mathcal{\hat{F}}({x}_{i}, \mathcal{W}_{i})
$$

$$
\frac{\partial E}{\partial {{x}_{l}}}=\frac{\partial E}{\partial {{x}_{L}}}\left((\prod_{i=l}^{L-1}\lambda_{i})+\frac{\partial {\sum_{i=l}^{L-1}\mathcal{\hat{F}}({x}_{i}\mathcal{W}_{i})}}{\partial {{x}_{l}}}\right) 
$$

公式说明：

1. 第一项会额外引进一个缩放因子$\prod_{i=l}^{L-1}\lambda_{i}$。对于一个极深的网络(L非常大)，如果$λ_i>1$，这个系数将会指数级大；如果 $λi<1$，这个系数将会变得指数级小并且消失，从而阻断从捷径反向传来的信号，并迫使它流向权重层，这将对优化造成困难。

2. 原始的identity skip connection被一个简单的缩放（$h(x_{l})=\lambda_{l}x_{l}$）代替。如果skip connection $h(x_l)$表示更复杂的变换（例如gating或者1x1卷积），公式的第一项变成了$\prod_{i=l}^{L-1}h'_{i}$，这里$h′$是$h$的导数。这个乘积也可能阻碍信息的反向传播并且阻碍训练训练过程。

**实验对比**：作者在CIFAR-10数据集上ResNet_v1-110实验。极深的ResNet_v1-110有54个两层残差单元(包含3x3卷积层)。假设f为恒等映射，这部分的实验中，和ResNet_v1一致，令f=ReLU。

分别对恒等跳跃连接支路（灰色箭头）：乘以一个常量缩放、sigmoid函数权重、1x1卷积、dropout，

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/v2-a90049141bc4cc28ff871f920bae8124_r.jpg width=75% />
</div>

- 常量缩放$\lambda$：常数比例相乘。

对于所有的捷径连接，设置λ=0.5。进一步考虑$F$的两种缩放情况：(1) F不进行缩放。(2) $f$用一个常量1−λ=0.5进行缩放，类似highway gating，但gate是固定的。(1)收敛的不好（测试错误率高于20%），(2)能够收敛，但是测试错误率（12.35%）比原始的ResNet-110高很多。表明当shortcut信号scaled down时，优化有困难。

- exclusive gating：1x1-sigmod函数训练权重，与两支路相乘。

和Highway Network一样，采用一个gating机制，作者考虑一个后面接sigmoid激活函数的gating函数$$g(x)=(W_gx+b_g)$$。在一个卷积网络中g(x)可以通过1x1卷积层实现。gating函数通过元素元素级别的乘法调节信号。

exclusive gating机制的影响是两面的，当1−g(x)接近1时，gated shortcut连接是十分接近于identity的，这有助于信息的传播；但是在这种情况下，g(x)接近0，并且抑制了函数FF。为了单独研究gating函数对于shortcut path的影响，我们接下来研究了没有exclusive gating机制。

- Shortcut-only gating：1x1-sigmod函数训练权重，与跳跃支路相乘。效果差

在这种情况下，函数F不进行缩放；shortcut path只用1−g(x)进行缩放）。bg的初始值对于这种情况仍然很重要。当bg的初始值是0时（所以1-g(x)的初始化的期望值为0.5），网络收敛到一个很烂的结果。这也是由于训练误差很高。

当bg的初始值是一个非常小的负数（very negatively biased 例如 -6），1−g(x)的值非常接近1并且shortcut连接接近于identity映射。因此结果（6.69%表1）是很接近ResNet_v1-110。

- 1x1 Convolutional shortcut：浅层好，深层差。

- Dropout shortcut：未收敛到好结果。

讨论：
如上图中灰色箭头所示，shortcut连接是信息传递最直接的路径。shortcut连接中的操作 (scale、gating、1××1 conv及 dropout) 会阻碍信息的传递，以致于优化困难。

值得注意的是1×1的卷积shortcut连接引入了更多的参数，本应该具有比恒等捷径连接更强大的表达能力。事实上，shortcut-only gating 和1×1的卷积涵盖了恒等捷径连接的解空间(即，他们能够以恒等捷径连接的形式进行优化)。然而，它们的训练误差比恒等捷径连接的训练误差要高得多，这表明了这些模型退化问题的原因是优化问题，而不是表达能力的问题

### 3、激活函数



<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/v2-df6cf78b3c0a90fd00a1d71884387498_r.jpg width=75% />
</div>

先验证了$h$和$F$公式为恒等映射的重要性，再验证二者相加后的输出为恒等映射，公式$f$如果也是一个恒等连接的重要性，通过调节激活函数 (`ReLU and/or BN`) 的位置，来使$f$是恒等映射原始的残差单元输出。

实验对比：

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/v2-5e5cf9e16a23a8a629165482bcd3de47_r.jpg width=75% />
</div>

1. BN after addition 效果比基准差，BN 层移到相加操作后面会阻碍信号传播，一个明显的现象就是训练初期误差下降缓慢。
2. ReLU before addition 这样组合的话残差函数分支的输出就一直保持非负，这会影响到模型的表示能力，而实验结果也表明这种组合比基准差。
3. Post-activation or pre-activation 原来的设计中相加操作后面还有一个 ReLU 激活函数，这个激活函数会影响到残差单元的两个分支，现在将它移到残差函数分支上，快捷连接分支不再受到影响。

注意：预激活方式又可以分为两种：只将 ReLU 放在前面，或者将 ReLU 和 BN都放到前面，根据实验结果可以看出 full pre-activation 的效果要更好。

预激活有两个方面的优点：1) [公式] 变为恒等映射，使得网络更易于优化；2)使用 BN 作为预激活可以加强对模型的正则化。

Ease of optimization 这在训练 1001 层残差网络时尤为明显。使用原来设计的网络在起始阶段误差下降很慢，因为 [公式] 是 ReLU 激活函数，当信号为负时会被截断，使模型无法很好地逼近期望函数；而使用预激活网络中的$f$是恒等映射，信号可以在不同单元直接直接传播。本文使用的 1001层网络优化速度很快，并且得到了最低的误差。

$f$为 ReLU 对浅层残差网络的影响并不大，如图 6-right 所示。本文认为是当网络经过一段时间的训练之后权值经过适当的调整，使得单元输出基本都是非负，此时$f$不再对信号进行截断。但是截断现象在超过 1000层的网络中经常发生。

Reducing overfitting，使用了预激活的网络的训练误差稍高，但却得到更低的测试误差，本文推测这是 BN 层的正则化效果所致。在原始残差单元中，尽管BN对信号进行了标准化，但是它很快就被合并到捷径连接(shortcut)上，组合的信号并不是被标准化的。这个非标准化的信号又被用作下一个权重层的输入。与之相反，本文的预激活（pre-activation）版本的模型中，权重层的输入总是标准化的。

## 训练与测试

略。

## 参考文献

[^01]: [黑暗星球-ResNet v2论文笔记-CSDN](https://blog.csdn.net/u014061630/article/details/80558661)

