# 调试工具 Windbg

摘要：Windbg是在windows平台下，强大的用户态和内核态调试工具。相比较于Visual Studio，它是一个轻量级的调试工具，所谓轻量级指的是它的安装文件大小较小，但是其调试功能，却比VS更为强大。它的另外一个用途是可以用来分析dump数据。
<!--more-->

# 调试工具 Windbg

## 1. 蓝屏问题查询

1. 我的电脑->右键管理->事件查看器->系统
2. 定位错误的信息`C:\WINDOWS\MEMORY.DMP`

## 2. Windbg安装
链接：[link](https://docs.microsoft.com/zh-cn/windows-hardware/drivers/debugger/debugger-download-tools)。

## 3. windbg分析MEMORY.DMP文件

1. `Open Crash Dump...`
2. 系统时间：System Uptime: 0 days 0:14:23.581，意思是0天(days)0小时14分23秒581毫秒时出现蓝屏了。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/20211220144112.png width=75% />
</div>

3. Probaly caused by（造成蓝屏可能的原因）

这个信息是相对比较重要的一个信息，如果你运气好的话，通过这个信息基本上可以看到导致蓝屏的驱动或者程序名称了，就像下图一样，初步的分析已经有了结果，Probaly caused by后面显示的是一个名为KiMsgProtect.sys的驱动文件导致蓝屏，这个文件就是恒信一卡通的一个关键驱动。因此蓝屏则很有可能和一卡通有关。
括号中驱动文件名后面的+号代表的是偏移地址，假如多个dmp文件的驱动文件名一样，且偏移地址也一样，则问题原因极有可能是同一个，这个偏移地址与汇编有关，这里不多做介绍。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/20211220144119.png width=75% />
</div>

其实，对于分析蓝屏dmp并不是每次运气都那么好，假如刚刚打开dmp文件未看到明确的蓝屏原因时，我们就需要借助一个命令来进一步分析dmp，这个命令就是：!analyze -v，这个命令能够自动分析绝大部分蓝屏原因。当初步分析没有结果时，可以使用该命令进一步分析故障原因，当然你也可以直接点击链接样式的!analyze -v来进行执行该命令，为了让大家更直观的看懂里面的信息，大家可以直接看图片中的注释信息。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/20211220144130.png width=75% />
</div>

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/20211220144138.png width=75% />
</div>

看了这么多信息之后，这个蓝屏dmp到底是怎么回事呢？根据dmp给出的信息，应该是：顾客上机0天(days)0小时14分23秒581毫秒时，一个名为PinyinUp.exe触发了KiMsgProtect.sys这个驱动的一个Bug，导致蓝屏。

那么PinyinUp.exe和KiMsgProtect.sys都是哪个厂商的？一般要知道这个信息，只能去用户的机器上找了，我去找了之后发现PinyinUp.exe是搜狗输入法的自动升级程序，KiMsgProtect.sys是恒信一卡通这个计费软件的驱动，所以这个dmp表示出来的意思看上去是搜狗拼音和恒信一卡通搞在一起，出了问题！当然排除方法很简单，把搜狗输入法的自动升级程序删除掉，再看看是否仍然有蓝屏问题发生就ok了！

学到这里，基本上已经可以分析绝大部分dmp文件了，但是分析蓝屏dmp要比较谨慎，对信息需要重新验证一次才更加保险，验证方法很简单，在WinDbg的命令输入框内，输入!process命令，就可以验证触发蓝屏的程序到底是否正确了。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/20211220144147.png width=75% />
</div>

5. 运行`$ !process`命令后得到的信息：

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/20211220144154.png width=75% />
</div>

至此，掌握以上几个简单的分析方法之后，基本上绝大多数dmp大家都可以独立分析了，当然WinDbg是个强大的工具，同时蓝屏的原因也有很多，如果想分析的足够准确，那么就只有多学多练，多去分析，因为WinDbg分析除了懂得几个命令之外，经验更加重要！

## 2. 参考

-  [01-吾王Excalibur-系统蓝屏日志DMP文件分析工具WinDbg及教程-CSDN](https://blog.csdn.net/qq_35583007/article/details/92793605?utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control)


