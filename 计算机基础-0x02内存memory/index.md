# 内存 Memory

摘要：内存(Memory)是计算机的重要部件，也称内存储器和主存储器，它用于暂时存放CPU中的运算数据，以及与硬盘等外部存储器交换的数据。
<!--more-->

# 内存 Memory
## 内存信息查看
### 1、内存占用
操作：`ctrl+alt+del` -> 任务管理器 -> 内存占用率。 注意：运行游戏，超过70%，还是需要添加或更换更大内存条。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/20211220142942.png width=75% />
</div>

### 2、内存卡槽情况

操作：`ctrl+alt+del` -> 任务管理器 -> 性能 -> 内存 -> 插槽。(双卡槽，已使用一个)

<div align=center>
	<img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/20211220142951.png width=75% />
</div>

### 3、内存容量
操作：`ctrl+alt+del` -> 任务管理器 -> 性能 -> 内存 -> 内存容量。(内存容量7.9G,已使用6.0G)

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/20211220143009.png width=75% />
</div>

### 4、CPU-Z查询内存信息
链接：[link](https://www.cpuid.com/softwares/cpu-z.html)。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/20211220143026.png width=75% />
</div>

内存信息：
- 内存类型DDR4 2666 8GB Samsung.

- DDR4不一定准确，更关注电压。

### 5、内存最大限制
- `cmd`->`$ wmic memphysical get maxcapacity`
- 结果/1024/1024 = 33554432/1024/1024 = 32.0 G

## 内存条
如果卡槽已满，只能更换内存条。如果想达到到最大内存，只能更换内存条。

### 1、双通道问题
如果可以的话，最好是买一个与电脑原先内存一样规格的内存条，然后开启双通道（一种让两个同时工作的内存条发挥更好效果的模式。
显卡类型（独立显卡or核心显卡）。

### 2、最大内存容量
最大内存容量决定你买多大的。

### 3、内存类型
内存类型决定你买什么类型。

### 4、demo暗夜精灵5GTX
注意：内存频率，越高频率，性能越好，两个不同频率的内存条同时使用，会使高频率的那个调频至低频率。
- DDR4 2666 Samsung. 
- 增加内存条只能买24G.
- 替换内存条可以买32G
