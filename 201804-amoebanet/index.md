# Regularized Evolution for Image Classifier Architecture Search

摘要：AmoebaNet是通过遗传算法的进化策略（Evolution）实现的模型结构的学习过程。
<!--more-->

# Regularized Evolution for Image Classifier Architecture Search[^01]

## 文献信息
| 信息 | 内容                                                         |
| ---- | ------------------------------------------------------------ |
| 日期 | 2018.04                                                      |
| 作者 | Esteban Real et al.                                          |
| 机构 | Google Brain                                                 |
| 来源 | AAAI 2019                                                    |
| 链接 | [Regularized Evolution for Image Classifier Architecture Search](https://arxiv.org/abs/1802.01548) |
| 代码 | [Code]()                                                     |

## 个人理解

><strong style="color:red;">问题:</strong> 网络结构优化；
>
><strong style="color:red;">方法:</strong> 遗传算法，引进年龄进化搜索最优结构；
>
><strong style="color:red;">结论:</strong> AmoebaNet取得了当时在ImageNet数据集上top-1和top-5的最高精度；
>
><strong style="color:red;">理解:</strong> 遗传算法理论；
>
><strong style="color:red;">优化：</strong>无；
---

## 背景知识

遗传算法进化理论：核心思想是优胜劣汰，为了公平，每次选择N名对象进行竞争，保证每次选择的都是随机的。

## 原理方法

AmoebaNet：遗传算法的进化策略（Evolution）实现的模型结构的学习过程。该算法的主要特点是在进化过程中引入了年龄的概念，使进化时更倾向于选择更为年轻的性能好的结构，这样确保了进化过程中的多样性和优胜劣汰的特点，这个过程叫做**年龄进化（Aging Evolution，AE）**。

作者为他的网络取名AmoebaNet，Amoeba中文名为变形体，是对形态不固定的生物体的统称，作者也是借这个词来表达AE拥有探索更广的搜索空间的能力。

### 1、搜索空间
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220508081348062.png width=75% />
</div>


仿照NASNet的思想，AmoebaNet也是学习两个Cell：

(1) Normal Cell：步长始终为1，因此不会改变Feature Map的尺寸，多个堆叠其目的是获得更大的模型容量，shortcut机制，即一个Normal Cell的输入来自上一层，另外一个输入来自上一层的上一层。

(2) Reduction Cell：步长为2，因此会将Feature Map的尺寸降低为原来的1/2。

在每个卷积操作中，我们需要学习两个参数：

1. 卷积操作的类型：类型空间参考NASNet。
2. 卷积核的输入：从该Cell中所有可以选择的Feature Map选择两个，每个Feature Map选择一个操作，通过合并这两个Feature Map得到新的Feature Map。最后将所有没有筛出的Feature Map合并作为最终的输出。上面所说的合并是单位加操作，因此Feature Map的个数不会改变。

举例说明一下这个过程，根据图中的跳跃连接，每个Cell有两个输入，对应图1右的0，1。那么第一个操作（红圈部分）选择0，1作为输入以及average池化和max池化作为操作构成新的Feature Map 2。接着第二个操作可以从（0，1，2）中选择两个作为输入，形成Feature Map 3，依次类推可以得到Feature Map 4，5，6，7。

最终AmoebaNet仅仅有两个变量需要决定，一个是每个Feature Map的卷积核数量 $F$ ，另一个是堆叠的Normal Cell的个数$N$ ，这两个参数作为了人工设定的超参数，作者也实验了$N$和$F$的各种组合。

### 2、Aging Evolution

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220508081858616.png width=75% />
</div>

暂时没有看懂，留在这里，以后再读。

1. 第1行是使用队列（queue）初始化一个population变量。在AE中每个变量都有一个固定的生存周期，这个生存周期便是通过队列来实现的，因为队列的“先进先出”的特征正好符合AE的生命周期的特征。population的作用是保存当前的存活模型，而只有存活的模型才有产生后代的能力。

2. 第2行的history是用来保存所有训练好的模型。

3. 第3行的作用是使用随机初始化的形式产生第一代存活的模型，个数正是循环的终止条件$P$ 。 $P$ 的值在实验中给出的个数有20，64，100三个，其中 $P=100$ 的时候得到了最优解。

4. while 循环中（4-7行）便是随机初始化一个网络，然后训练并在验证集上测试这个网络的精度，最后将网络的架构和精度保存到population和history变量中。这里所有的模型评估都是在CIFAR-10上完成的。首先注意保存的是架构而不是模型，所以保存的变量的内容不会很多，因此并不会占用特别多的内存。其次由于population是一个队列，所以需要从右侧插入。而history插入变量时则没有这个要求。

5. 第9行的第二个while循环表示的是进化的时长，即不停的向history中添加产生的优秀模型，直到history中模型的数量达到 $C$ 个。 $C$ 的值越大就越有可能进化出一个性能更为优秀的模型，我们也可以选择在模型开始收敛的结束进化。在作者的实验中 $C=2000$ 。

6. 第10行的sample变量用于从存活的样本中随机选取 $S$ 个模型进行竞争，第三个while循环中的代码（11-15行）便是用于随机选择候选父带。

7. 第16行代码是从这$S$个模型只有精度最高的产生后代。这个有权利产生后代的变量命名为parent。论文实验中$parent$的值设定的值有2，16，20，25，50，其中效果最好的值是25。

8. 第17行是使用变异（mutation）操作产生父代的子代，变量名是child。变异的操作包括随机替换卷积操作（op mutation）和随机替换输入Feature Map（hidden state mutation），如图所示。在每次变异中，只会进行一次变异操作，亦或是操作变异，亦或是输入变异。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220508083234809.png width=75% />
</div>


AmoebaNet的变异操作：(上)Hidden State Mutation改变模型的输入Feature Map；(下)Op Mutation改变一个卷积操作

9. 第18-20行依次是训练这个子代网络架构并将它依次插入population和history中。
10. 第21-22行是从population顶端移除最老的架构，这一行也是AE最核心的部分。另外一种很多人想要使用的策略是移除效果最差的那个，这个方法在论文中叫做Non Aging Evolution（NAE）。作者这么做的动机是如果一个模型效果足够好，那么他有很大概率在他被淘汰之前已经在population中留下了自己的后代。如果按照NAE的思路淘汰差样本的话，population中留下的样本很有可能是来自一个共同祖先，所以AE的方法得到的架构具有更强大的多样性。而NAE得到的架构由于多样性非常差，使得架构非常容易陷入局部最优值。这种情况在遗传学中也有一个名字：近亲繁殖。
11. 最后一行代码是从所有训练过的模型中选择最好的那个作为最终的输出。

### 3、AmoebaNet网络结构

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220508081953849.png width=75% />
</div>


两个要手动设置的参数，一个参数是连续堆叠的Normal Cell的个数 N ，另外一个是卷积核的数量。在第一个Reduction之前卷积核的数量是 F，后面每经过一次Reduction，卷积核的数量 Fx2 。这两个参数是需要人工设置的超参数。实验的时候，N=6，F=190。

## 实验结果

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220508082157183.png width=75% />
</div>


实验结果表明，当AmoebaNet的参数数量（N=6，F=190）达到了NASNet以及PNASNet的量级（80MB+)时，AmoebaNet和其它两个网络在ImageNet上的精度是非常接近的。虽然AmoebaNet得到的网络和NASNet以及PNASNet非常接近，但是其基于AE的收敛速度是要明显快于基于强化学习的收敛速度的。

而最好的AmoebaNet的参数数量达到了469M时，AmoebaNet-A取得了目前在ImageNet上最优的测试结果。但是不知道是得益于AmoebaNet的网络结构还是其巨大的参数数量带来的模型容量的巨大提升。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/image-20220508082416195.png width=75% />
</div>

Aging Evolution（AE，进化算法）和RL（Reinfrocement Learning，强化学习）结果相等，但AE的收敛快于RL，也不说AE优于RL。

## 参考文献

[^01]: [大师兄-AmoebaNet详解-知乎](https://zhuanlan.zhihu.com/p/57489362)


