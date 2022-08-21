# SqueezeNet, AlexNet-level accuracy with 50x fewer parameters and <0.5MB model size

摘要：轻量级网络SquezeNet，亮点在于Fire结构，先压缩再扩展。
<!--more-->

# SqueezeNet: AlexNet-level accuracy with 50x fewer parameters and <0.5MB model size[^01][^02]

## 文献信息
| 信息 | 内容                                                         |
| ---- | ------------------------------------------------------------ |
| 日期 | 2016. 02                                                     |
| 作者 | Forrest N. Iandola et al.                                    |
| 机构 | DeepScale & UC Berkeley 2Stanford University                 |
| 来源 | ICLR 2017                                                    |
| 链接 | [SqueezeNet: AlexNet-level accuracy with 50x fewer parameters and <0.5MB model size](https://arxiv.org/abs/1602.07360) |
| 代码 | [forresti/SqueezeNet](https://github.com/forresti/SqueezeNet) |

## 个人理解
><strong style="color:red;">问题:</strong> 轻量级网络设计；
>
><strong style="color:red;">方法:</strong> 文章提出了Fire结构；
>
><strong style="color:red;">结论:</strong> SqueezeNet在ImageNet上实现了AlexNet级别的精度，参数减少了50倍。此外，通过模型压缩技术，能够将SqueezeNet压缩到小于0.5MB；
>
><strong style="color:red;">理解:</strong> “压缩+扩展”思想。
>
><strong style="color:red;">优化：</strong>参考[^02]
>
>1. SqueezeNet的侧重的应用方向是嵌入式环境，目前嵌入式环境主要问题是实时性。SqueezeNet的通过更深的深度置换更少的参数数量虽然能减少网络的参数，但是其丧失了网络的并行能力，测试时间反而会更长，这与目前的主要挑战是背道而驰的；
>2. 论文的题目非常标题党，虽然纸面上是减少了50倍的参数，但是问题的主要症结在于AlexNet本身全连接节点过于庞大，50倍参数的减少和SqueezeNet的设计并没有关系，考虑去掉全连接之后3倍参数的减少更为合适。
>3. SqueezeNet得到的模型是5MB左右，0.5MB的模型还要得益于Deep Compression。虽然Deep Compression也是这个团队的文章，但是将0.5这个数列在文章的题目中显然不是很合适。
---

## 背景知识

在深度学习崭露头角时候，很多研究都关注如何提高网络的准确率，而SqueezeNet则是早期开始关注轻量化网络的研究之一。论文的初衷是通过优化网络的结构，在与当前流行网络的准确率相差不大的情况下，大幅减少模型的参数。

小型 CNN 的优势：

- 在分布式平台训练时，如果采用较小的 CNN,意味各个分布式子系统之间通讯量会减少，有助于提高训练性能。
- 未来的自动驾驶汽车都有 OTA,也就是在线升级的功能，较小的 CNN 有助于减少云端的传输压力，因为较小的 CNN数据量更少，节约了用户下载时间，也节省了流量。
- 较小的 CNN 更容易部署在 FPGA 这样内存受限的硬件上。

## 原理方法

### 1、压缩策略

1. 将 3x3 卷积替换成 1x1 卷积：通过这一步，一个卷积操作的参数数量减少了9倍；
2. 减少 3x3 卷积的通道数：一个3x3 卷积的计算量是 3x3xMxN （其中 M,N分别是输入Feature Map和输出Feature Map的通道数），作者任务这样一个计算量过于庞大，因此希望将M,N减小以减少参数数量；
3. 将降采样后置：作者认为较大的Feature Map含有更多的信息，因此将降采样往分类层移动。注意这样的操作虽然会提升网络的精度，但是它有一个非常严重的缺点：即会增加网络的计算量。

### 2、Fire模块

![image-20220507104201154](C:/Users/ly/AppData/Roaming/Typora/typora-user-images/image-20220507104201154.png)

Fire模块：Squeeze部分和Expand部分组成，（注意区分SENet的区别）。
- Squeeze部分是1x1卷积；
- Expand部分是由1x1卷积和3x3卷积的cancat；

在Fire模块中，Squeeze部分 1x1 卷积的通道数记做$s_{1 \times 1}$，Expand部分1x1卷积和3x3卷积的通道数分别记做$e_{1 \times 1}$和$e_{3 \times 3}$。作者建议 $s_{1 \times 1} < e_{1 \times 1} + e_{3 \times 3}$，这么做相当于在两个3x3卷积的中间加入了瓶颈层，作者的实验中的一个策略是$s_{1 \times 1} = \frac{e_{1 \times 1}}{4} + \frac{e_{3 \times 3}}{4}$ ，其中$s_{1 \times 1} = 3$，$ e_{1 \times 1} = e_{3 \times 3} = 4$。

### 3、SqueezeNet的网络架构

![image-20220507105356078](C:/Users/ly/AppData/Roaming/Typora/typora-user-images/image-20220507105356078.png)

网络结构：无short-cut的SqueezeNet，short-cut的SqueezeNet，右侧是short-cut和多Feature Map个数的卷积的。

其他细节：激活函数为ReLU；添加了fire9之后接了一个rate为0.5的dropout。

![image-20220507105601037](C:/Users/ly/AppData/Roaming/Typora/typora-user-images/image-20220507105601037.png)

## 实验结果

![image-20220507105725303](C:/Users/ly/AppData/Roaming/Typora/typora-user-images/image-20220507105725303.png)

![image-20220507105739667](C:/Users/ly/AppData/Roaming/Typora/typora-user-images/image-20220507105739667.png)

![image-20220507105815231](C:/Users/ly/AppData/Roaming/Typora/typora-user-images/image-20220507105815231.png)

## 参考文献
[^01]: [VincentLee-SqueezeNet/SqueezeNext简述 | 轻量级网络-知乎](https://zhuanlan.zhihu.com/p/153103125)
[^02]: [大师兄-SqueezeNet详解-知乎](https://zhuanlan.zhihu.com/p/49465950)

