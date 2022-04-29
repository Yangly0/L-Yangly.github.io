# Deep Residual Learning for Image Recognition

摘要：残差学习的目的是让模型的内部结构至少有恒等映射的能力，以保证在堆叠网络的过程中，网络至少不会因为继续堆叠而产生退化!
<!--more-->

# Deep Residual Learning for Image Recognition

## 文献信息
| 信息 | 内容                                                         |
| ---- | ------------------------------------------------------------ |
| 日期 | 2015.12                                                      |
| 作者 | [Kaiming He](http://kaiminghe.com/) et al.                   |
| 机构 | Microsoft Research                                           |
| 来源 | CVPR2016 最佳论文 (Computer Vision and Pattern Recognition)  |
| 链接 | [Deep Residual Learning for Image Recognition](https://arxiv.org/abs/1512.03385) |
| 代码 | [Code]()                                                     |

## 个人理解

><strong style="color:red;">问题:</strong> 网络深度问题
>
>1. 梯度爆炸和消散问题：由于随着层数的增多，在网络中反向传播的梯度会随着连乘变得不稳定，变得特别大或者特别小，从而梯度爆炸或消散。为了克服梯度问题也诞生了许多的解决办法，如归一化BatchNorm，激活函数ReLu，使用Xaiver初始化，随机梯度下降（SGD）等。
>2. 网络的退化问题：即网络深度增加时，网络准确度出现饱和，甚至出现下降。注意：并不是过拟合问题，因为深层网络的训练集和验证集的误差都比较高，而过拟合是训练集误差低验证集误差高。
>
><strong style="color:red;">方法:</strong> 残差学习解决退化问题；
>
><strong style="color:red;">结论:</strong> ResNet在ILSVRC和COCO 2015取得5项第一；
>
><strong style="color:red;">理解:</strong> 
>
>1. 网络退化问题：理论上来说，堆叠网络，哪怕这些增加层什么也不学习，仅仅复制浅层网络的特征，即恒等映射（Identity mapping），在这种极端情况下，深层网络应该和浅层网络性能一样，也不应该退化，可能训练方法有问题，才使得深层网络很难去找到一个好的参数。
>2. 残差学习，从特征的学习$H(x)$映射到残差的学习$F(x)$，对应的特征$H'(x) = F(x) + x$，残差学习更容易学习：（1）当残差为0时，堆叠层相当于恒等映射，网络不会退化；（2）实际上残差并不为0，会使得网络在输入特征上学到新特征，达到更好的性能。
>
><strong style="color:red;">优化：</strong>
>
>1. 为什么残差学习相对容易小？从梯度入手来$y' = 1 + F'(x)$。
>2. 不同理解：（1）即使BN过后梯度的模稳定在了正常范围内，但**梯度的相关性实际上是随着层数增加持续衰减的**。而经过证明，ResNet可以有效减少这种相关性的衰减；（2）跳连接相加可以实现不同分辨率特征的组合；（DenseNet，跳接组合concat更多分辨率的特征，模型的效果会更好？）（3）引入跳接实际上让模型自身**有了更加“灵活”的结构**，即在训练过程本身，模型可以选择在每一个部分是“更多进行卷积与非线性变换”还是“更多倾向于什么都不做”，抑或是将两者结合，[link](https://www.zhihu.com/question/64494691/answer/786270699)。
>3. 问题1：既然非常深的模型有这样那样的劣势，那么为什么不直接减少层数？给我的感觉ResNet更多的是为了保证更深的模型不会变得更糟。或者说，减到多少层是一个比添加Residual block更糟心更费时的过程？（模型越深越能拟合复杂的表达，浅层网络达不到要求）
>4. 问题2：加了shortcut以后，back propagation的时候如何更新参数？求导结果上加了一个常数。
>5. 问题3：已有的神经网络很难拟合潜在的恒等映射函数$H(x)=x$？有激活函数，会卡掉一部分输入的值。
>6. 问题4：恒等映射了，那就是后面的层理想状态下输入输出没有改变，那还要这些层干嘛呢？恒等是加在那些层的输出上的，输出是卷积和恒等映射的共同结果？
>7. 问题5：有没有可能训练结束后发现有一个Residual Block作用相当于恒等映射，而之后的层并不是，这样的话是不是就能把这个block去掉了？
>8. 问题6：普通残差block至少包含两层卷积，一层卷积呢？“先求和再激活”还是“先激活再求和”，前者无意义，后者有意义。如果一层卷积，先求和再激活就没有意义，后者就是有意义的。
---

## 背景知识

深度卷积神经网络：特征“level”可以通过堆叠层数（深度）来丰富低、中和高级特征。

深度随之带来的问题1：梯度消失和爆炸，阻碍收敛。权重初始化、归一化和随机梯度下降都能有效解决该问题。

深度随之带来的问题1：网络退化，随着网络深度的增加，精度会达到饱和（这可能并不奇怪），然后迅速退化。这种退化并不是由过度拟合引起的，向适当深度的模型中添加更多层会导致更高的训练误差，训练精度的降低表明并非所有系统都同样容易优化。

Residual Representations：

- VLAD通过残差向量相对于字典进行编码；
- Fisher向量可以表示为VLAD的概率版本；

Shortcut Connections：

- 中间层直接连接到辅助分类器，用于处理消失/爆炸梯度；
- 通过快捷连接实现的层响应、梯度和传播误差的中心化方法；

## 原理方法

### 1、思想
借鉴VLAD（残差思想）和[Highway Network](https://arxiv.org/pdf/1505.00387.pdf)（跨层连接思想，相加$y=F(x)+Wx$），提出了shortcut connection结构。

$$
y = F(x) + x \\ 
F(x) = Wx + b
$$
**恒等映射（identity mappings）**：如果F(x) = 0, y=x, 即为恒等映射（H(x) = x）。事实上，当层数太深了，F(x)趋近与0，使得无论x为何值时，F(x)总为0，y = x，不会出现退化现象。

**为什么残差容易学习？**：值得一提的是，假如优化目标函数是逼近一个恒等映射, 而不是0映射， 那么学习找到对恒等映射的扰动会比重新学习一个映射函数要容易。更通俗来说对于某层特征$y_l=F_l(F_{l-1}(x_{l-1}) + Wx_{l-1})+Wx_l$，当网络梯度$y'=1+F'(x)$，即残差梯度不会恰巧都为-1，而且就算很小，有1的存在也不会导致梯度消失。

ResNet改变目标函数，目标值y和x的差值，即所谓的残差$F(x) := y-x$。因此，后面的训练目标就是要将残差结果逼近于0，使到随着网络加深，准确率不下降，使输入x近似于输出y，以保持在后面的层次中不会造成精度下降。

### 2、结构

残差单元：普通残差块（浅层）和瓶颈残差块（深层），

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220424000959498.png width=75% />
</div>

短路链接：对于输入输出一致，直接相加；对于输入输出不一致的处理细节
- 空间上不一样，跳连部分给x添加一个线性映射$y = F(x) + Wx$，即stride=2是来解决空间
- 维度上不一样，（1）补零填充；（2）1x1卷积升维。两种都可行，主要是会不会增加参数，从而影响计算量。

网络结构：

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/v2-1dfd4022d4be28392ff44c49d6b4ed94_r.jpg width=75% />
</div>

## 参考文献
[^01]: [会哭泣的猫-ResNet网络详细解析-CSDN](https://blog.csdn.net/qq_41760767/article/details/97917419?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)
[^02]: [小小将-你必须要知道CNN模型：ResNet-知乎](https://zhuanlan.zhihu.com/p/31852747)

