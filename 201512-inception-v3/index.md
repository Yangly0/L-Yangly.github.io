# Rethinking the Inception Architecture for Computer

摘要：针对计算效率方面的作用，提出了分解卷积等思想。	
<!--more-->

# Rethinking the Inception Architecture for Computer

## 文献信息
| 信息 | 内容                                                         |
| ---- | ------------------------------------------------------------ |
| 日期 | 2015.12                                                 |
| 作者 | Christian Szegedy et al.                                                   |
| 机构 | Google Inc                                                     |
| 来源 | arXiv                                                     |
| 链接 | [Rethinking the Inception Architecture for Computer](https://arxiv.org/abs/1512.00567) |
| 代码 | [Code]()                                                     |

## 个人理解

><strong style="color:red;">问题:</strong> Inception v1的计算量，尤其是通道上，计算量和channel的平方成正比；
> 
><strong style="color:red;">方法:</strong> 文章提出了网络设计准则，分解卷积核设计，辅助分类支路，分辨率下降优化、标签平衡、高分辨率输入等技巧；
> 
><strong style="color:red;">结论:</strong> ILSVRC 2012，21.2%top-1错误率和5.6%top-5错误率，四模型融合平均17.3%top-1错误率和3.5%top-5错误率；
> 
><strong style="color:red;">理解:</strong> 核心思想是可分解卷积核和正则化，对Inception结构维度减少和并行结构。
> 
><strong style="color:red;">优化：</strong>。
---

## 原理方法

### 1、设计准则
1. 避免代表性（表现上）瓶颈，尤其是在网络早期。
   - 通常，在达到任务的最终表示之前，表示大小应该从输入到输出逐渐减小。
   - 维度信息仅提供对信息内容的粗略估计。
2. 更高维的表示更容易在网络内本地处理。
   - 增加卷积网络中每个图块的激活次数可以实现更多解开的特征。由此产生的网络将训练得更快。
3. 空间聚合可以在较低维度的嵌入上完成，而不会损失太多或任何表示能力。
   - 例如，在执行更分散（例如 3 × 3）卷积之前，可以在空间聚合之前减少输入表示的维度，而不会产生严重的不利影响。
   - 我们假设其原因是相邻单元之间的强相关性导致降维期间信息丢失少得多，如果输出用于空间聚合上下文。
   - 鉴于这些信号应该很容易压缩，降维甚至可以促进更快的学习。
4. 平衡网络的宽度和深度。

### 2、分解卷积核尺寸

第一种方法：分解为对称的小的卷积核，VGG中的思想，将5x5的卷积核替换成2个3x3的卷积核。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427154335337.png width=25% />  <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427154340217.png width=25% /> 
</div>



第二种方法：分解为不对称的卷积核，将nxn的卷积核替换成 1xn 和 nx1 的卷积核堆叠，计算量又会降低。但是第二种分解方法在大维度的特征图上表现不好，在特征图12-20维度上表现好。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427154450936.png width=25% />  <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427154529752.png width=25% /> 
</div>



不对称分解方法有几个优点：

- 节约了大量的参数
- 增加一层非线性，提高模型的表达能力
- 可以处理更丰富的空间特征，增加特征的多样性

### 3、使用辅助分类器

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427154651191.png width=75% />
</div>

在第一篇论文中GoogLeNet中就使用了辅助分类器，使用了2个，但前期不会加速拟合，后期的准确率会有些微的提高。其优势：

- 把梯度有效的传递回去，不会有梯度消失问题，加快了训练。
- 中间层的特征也有意义，空间位置特征比较丰富，有利于提成模型的判别力。

### 4、高效降低特征图尺寸的方式

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427154914072.png width=35% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427154807784.png width=35% />
</div>

设计准则的第一条，就是避免表达瓶颈。那么传统的卷积神经网络的做法，当有pooling时（pooling层会大量的损失信息），会在之前增加特征图的厚度（就是双倍增加滤波器的个数），通过这种方式来保持网络的表达能力，但是计算量会大大增加。

优化：分离两个通道，一个是卷积层，一个是pooling层，两个通道生成的特征图大小一样，concat在一起即可。

### 5、Inception v3网络结构

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427154209135.png width=50% />
</div>

Inception v2不同的是，作者将7x7卷积分解成了三个3x3卷积。网络中有三个Inception模组，三个模组的结构分别采用 图5、6、7三种结构。inception模块中的gird size reduction方法采用的是图10结构。
我们可以看到，网络的质量与第二节说的准则有很大关系。尽管我们的网络深达42层，但我们的计算量仅仅是GoogLeNet的2.5倍，并且，它比VGG更高效。

### 6、Label Smoothing模型正则[^02]

针对cross-entropy loss的缺点，即必须使得对应真值的logit过大。原因在于softmax公式： $p(k|x) = \frac{exp(z_k)}{\sum_{i=1}^{K}exp(z_i)}$ ，要使 $p(k|x) = 1$ 需使得$exp(z_k) = \sum_{i=1}^{K}exp(z_i)$ 即 $\sum_{i=1,i!=k}^{K}exp(z_i) = 0$，当然这个是理想情况，一般导致的结果就是$(z_i)_{i!=k} << z_k$。

1. 这可能会导致过度拟合：如果模型学会为每个训练示例为真实标签分配完整概率，则不能保证泛化。
2. 它鼓励最大logit与所有其他logit之间的差异变大，而这个最大logit和所有其他logit之间的差异变大，并且与有界梯度相结合，降低了模型的适应能力。 直观地说，这是因为模型对其预测过于自信。

作者提出了一个正则化方法LSR(label-smoothing regularization)：对比之前的真值分布$p(k|x) = \delta_{k,y}$，LSR提出的真值分布是：

$$
q^{\prime}(k)=(1-\epsilon) \delta_{k, y}+\frac{\epsilon}{K}
$$

因为$\epsilon$是一个小的正数，所以对应真值($\delta_{k, y} = 1$)的大小是$q^{\prime}(k)=1 - \frac{K-1}{K}\epsilon$ ，可见基本相当于1减掉一个小数，防止无限趋于0，对应非真值($\delta_{k, y} = 0$)的大小是 $q^{\prime}(k)=\frac{\epsilon}{K}\epsilon$ ，是比$\epsilon$更小的一个数，防止其为0罢了。

### 7、在低分辨率输入情况下的性能

研究分辨率的影响是为了搞清楚：高分辨率是否有助于性能的提升，能提高多少？

一个简单方法是在较低分辨率输入的情况下减少前两层的步幅，或者简单地删除网络的第一个池化层。

但作者采用了三种分辨率的图像作为输入。三种情况的计算量是几乎相同的。此外，表 2 的这些结果表明，可以考虑在 R-CNN [5] 上下文中为较小的对象使用专用的高成本低分辨率网络。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427155106542.png width=75% />
</div>


## 实验结果

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427155714507.png width=50% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427155735269.png width=50% />
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220427155745538.png width=50% />
</div>

## 参考文献

[^01]: [chairon-Inception V3-CSDN](https://blog.csdn.net/chairon/article/details/119445971)

[^02]: [Michael-深入解读Inception V3-知乎](https://zhuanlan.zhihu.com/p/50751422)

