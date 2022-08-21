# You Only Look Once - Unified, Real-Time Object Detection

摘要：Yolov1的思想是整张图作为网络的输入，直接在输出层回归边界框的位置和类别；
<!--more-->

# You Only Look Once: Unified, Real-Time Object Detection

## 0x01 文献信息

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/imgs/20220821102318.png width=75% />
</div>

| 信息 | 内容                                                         |
| ---- | ------------------------------------------------------------ |
| 日期 | Jun 2015                                                     |
| 作者 | Joseph Redmon et al.                                         |
| 机构 | xxxxxxxx                                                     |
| 来源 | arXiv                                                        |
| 链接 | [https://arxiv.org/abs/1506.02640](https://arxiv.org/abs/1506.02640) |
| 代码 | [Code]()                                                     |

## 0x02 个人理解
><strong style="color:red;">问题:</strong> 两阶段R-CNN系列网络的缺陷，生成候选框问题，再进行分类器对候选框划分，网络速度慢且难于优化因为需要分开训练；
>
><strong style="color:red;">方法:</strong> 文章提出了一阶段网络，回归坐标和类别思想；
>
><strong style="color:red;">结论:</strong> 速度达到45FPS和155FPS，精度提升两倍；
>
><strong style="color:red;">理解:</strong> 一阶段的开山鼻祖，回归坐标和类别，以网格划分图片，对每个网格来预测相应对象；
>
>- YOLO对相互靠的很近的物体，还有很小的群体 检测效果不好，这是因为一个网格中只预测了两个框，并且只属于一类。
>- 由于损失函数的问题，定位误差是影响检测效果的主要原因。尤其是大小物体的处理上，还有待加强。
>- 对测试图像中，同一类物体出现的新的不常见的长宽比和其他情况，泛化能力偏弱。
---

## 0x03 背景知识

目标检测：确定物体的类别和位置；

传统的思路：基于滑窗的思想，再对窗口分类，DPM思路；

两阶段网络：R-CNN系列，先生成候选框，再对候选框分类和微调坐标；

## 0x04 原理方法

### (1) 原理[^01]

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/imgs/20220819000358.png width=75% />
</div>

1. 将一幅图像分成SxS个网格(grid cell)，如果某个object的中心落在这个网格中，则这个网格就负责预测这个object；

2. 每个网格要预测B个**边界框**bounding box，每个bounding box除了要回归自身的位置之外，还要附带预测一个confidence值；
   这个confidence代表了所预测的box中含有object的置信度和这个box预测的有多准两重信息，其值是这样计算的：$Pr(Object) * IOU^{truth}_{pred}$；

   其中，如果有object落在一个grid cell里，第一项取1，否则取0。 第二项是预测的bounding box和实际的groundtruth之间的IoU值；

3. 每个bounding box要预测(x, y, w, h)和confidence共5个值，每个网格还要预测一个类别信息，记为C类。则SxS个网格，每个网格要预测B个bounding box还要预测C个categories。输出就是S x S x (5*B+C)的一个tensor。 
   注意：class信息是针对每个网格的，confidence信息是针对每个bounding box的。

实例：在PASCAL VOC中， 图像输入为448x448，取S=7，B=2，一共有20个类别(C=20)。则输出就是7x7x30的一个tensor。

### (2) 网络结构

网络结构借鉴了 GoogLeNet：24个卷积层，2个全链接层。（用1×1 reduction layers 紧跟 3×3 convolutional layers 取代Goolenet的 inception modules ）

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/imgs/20220820221324.png width=75% />
</div>
网络输出:每个grid有30维，这30维中，8维是回归box的坐标，2维是box的confidence，还有20维是类别。 
其中坐标的x,y用对应网格的offset归一化到0-1之间，w,h用图像的width和height归一化到0-1之间；

### (3) 损失函数

平方和误差损失: 坐标预测 + 置信度预测 + 类别预测；
$$
\begin{aligned}

loss & = \lambda_{coord} \sum^{S^2}_{i=0}\sum^B_{j=0}\prod^{obj}_{ij}{\{(x_i - \hat{x}_i)^2 + (y_i - \hat{y}_i)^2\}} \\ 
& \qquad + \lambda_{coord} \sum^{S^2}_{i=0}\sum^B_{j=0}\prod^{obj}_{ij}{\{(\sqrt{w_i} - \sqrt{\hat{w}_i})^2 + (\sqrt{H_i} - \sqrt{\hat{H}_i})^2\}}  \\

& \qquad +  \sum^{S^2}_{i=0}\sum^B_{j=0}\prod^{obj}_{ij} (C_i - \hat{C_i})^2 
 + \lambda_{noobj}  \sum^{S^2}_{i=0}\sum^B_{j=0}\prod^{noobj}_{ij} (C_i - \hat{C_i})^2 \\

& \qquad +   \sum^{S^2}_{i=0} \prod^{obj}_{ij} \sum_{c \in classes} (p_i(c) - \hat{p_i}(c))^2

\end{aligned}
$$

- 只有当某个网格中有object的时候才对classification error进行惩罚;
- 只有当某个box predictor对某个ground truth box负责的时候，才会对box的coordinate error进行惩罚，而对哪个ground truth box负责就看其预测值和ground truth box的IoU是不是在那个cell的所有box中最大;

损失函数的设计目标就是让坐标（x,y,w,h），confidence，classification 这个三个方面达到很好的平衡。简单的全部采用了sum-squared error loss来做这件事会有以下不足：

a) 8维的localization error和20维的classification error同等重要显然是不合理的； b) 如果一个网格中没有object（一幅图中这种网格很多），那么就会将这些网格中的box的confidence push到0，相比于较少的有object的网格，这种做法是overpowering的，这会导致网络不稳定甚至发散。 

解决方案如下：

- 更重视8维的坐标预测，给这些损失前面赋予更大的loss weight, 记为$\lambda_{coord}$，在pascal VOC训练中取5。
- 对没有object的bbox的confidence loss，赋予小的loss weight，记为$\lambda_{noobj}$，在pascal VOC训练中取0.5。
- 有object的bbox的confidence loss和类别的loss的loss weight正常取1。
- 对不同大小的bbox预测中，相比于大bbox预测偏一点，小box预测偏一点更不能忍受。而sum-square error loss中对同样的偏移loss是一样。 为了缓和这个问题，作者用了一个比较取巧的办法，就是将box的width和height取平方根代替原本的height和width。因为small bbox的横轴值较小，发生偏移时，反应到y轴上的loss比big box要大。

一个网格预测多个bounding box，在训练时我们希望每个object（ground true box）只有一个bounding box专门负责（一个object 一个bbox）。具体做法是与ground true box（object）的IOU最大的bounding box负责该ground true box(object)的预测。这种做法称作bounding box predictor的specialization(专职化)。每个预测器会对特定（sizes, aspect ratio or classed of object）的ground true box预测的越来越好。（个人理解：IOU最大者偏移会更少一些，可以更快速的学习到正确位置）

REF: https://github.com/abeardear/pytorch-YOLO-v1/blob/master/yoloLoss.py

## 0x05 训练测试

预训练分类网络：在 ImageNet 1000-class competition dataset上预训练一个分类网络，这个网络前20个卷机网络+average-pooling layer+ fully connected layer （此时网络输入是224x224）。


训练检测网络：转换模型去执行检测任务，《Object detection networks on convolutional feature maps》提到说在预训练网络中增加卷积和全链接层可以改善性能。在他们例子基础上添加4个卷积层和2个全链接层，随机初始化权重。检测要求细粒度的视觉信息，所以把网络输入也又224x224变成448x448。

测试过程：

1. Resize成448*448，图片分割得到7x7网格(cell)
2. CNN提取特征和预测：卷积部分负责提特征。全链接部分负责预测：a) 7x7x2=98个bounding box(bbox) 的坐标$x_{center},y_{center},w,h$ 和是否有物体的conﬁdence 。 b) 7x7=49个cell所属20个物体的概率。
3. 过滤bbox（通过nms），得到目标；


在test的时候，先根据置信度选择含目标的网格区域（即每个cell只选最大概率的那个预测框，且阈值大于0.1），对应最大的分类概率作为该网格区域的类别，分别计算对应的坐标和得分，即每个网格预测的class信息和bounding box预测的confidence信息相乘，就得到每个bounding box的class-specific confidence score: 
$$
Pr(Class_i | Object) * Pr(Object) * IOU^{true}_{pred} = Pr_{class_i} * IOU^{true}_{pred}
$$

其中，第一项就是每个网格预测的类别信息，第二、三项就是每个bounding box预测的confidence。这个乘积即encode了预测的box属于某一类的概率，也有该box准确度的信息。

根据每个box的class-specific confidence score以后，设置阈值，滤掉得分低的boxes，对保留的boxes进行NMS处理，就得到最终的检测结果。

## 0xFF 参考文献

[^01]: [Fighting_1997-yolov1详解-CSDN](https://blog.csdn.net/frighting_ing/article/details/123450918)

