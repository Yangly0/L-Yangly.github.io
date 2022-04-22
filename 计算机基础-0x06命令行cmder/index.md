# 命令行 Cmder

摘要：Cmder是一个软件包，它被创建出来替代极不满没有漂亮的控制台模拟器的Windows上。
<!--more-->

# 命令行 Cmder
## 介绍
Cmder是由于Windows上缺少漂亮的控制台模拟器而完全出于沮丧而创建的一个软件界面。它基于出色的软件，并采用了Monokai配色方案和自定义的提示布局。

## 下载
下载地址：[https://cmder.net/](https://cmder.net/)，Mini和Full区别在于Git，按个人喜好下载。

## 安装
1. 解压
2. (可选) 配置环境变量
   （我的电脑->右键/属性->高级系统设置->环境变量->系统变量/Path/编辑->新建->`D:\cmder`)
3. `win+R`，运行Cmder(Cmder.exe)

## 配置
### 1、右键菜单
```bash
$ cmder.exe /REGISTER ALL
```

### 2、中文设置和中文乱码问题
右击cmd窗口，点击setting。选择General，修改Interface language。
右击cmd窗口，点击setting。在Start-up下的environment中加入: set LANG=zh_CN.UTF8，解决乱码。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/imgs/20211220142811.png width=75% />
</div>

### 3、λ改为$
修改文件： `D:\cmder\vendor\clink.lua`，如下

```lua
    -- local cmder_prompt = "\x1b[1;32;40m{cwd} {git}{hg}{svn} \n\x1b[1;39;40m{lamb} \x1b[0m"
    local cmder_prompt = "\x1b[1;32;40m{cwd}{git}{hg}{svn}\x1b[1;39;40m{lamb} \x1b[0m"
    
    -- local lambda = "λ"
    local lambda = "$"

    cmder_prompt = string.gsub(cmder_prompt, "{cwd}", verbatim(cwd))
    -- if env ~= nil then
    --     lambda = "("..env..") "..lambda
    -- end
    if env ~= nil then
        cmder_prompt = "\x1b[1;32;40m("..env..")\x1b[1;32;40m"..cmder_prompt
    end
    clink.prompt.value = string.gsub(cmder_prompt, "{lamb}", verbatim(lambda))
```

### 4、默认启动和启动设置
修改默认启动，cmder，powershell，WSL(window子系统)。
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/20211220142618.png width=75% />
</div>

添加命令 `cmd /k ""%ConEmuDir%\..\init.bat" "`
<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/20211220142627.png width=75% />
</div>

### 5、字体，背景，主题，透明度
字体设置：在setting->通用->字体->大小（20）

背景设置：在setting->通用->背景->路径（你的图片位置）

主题设置：在 setting->功能特性->颜色 ->方案（Monokai）

调节背景透明度： 在 setting->功能特性->透明度->Alpha透明度 。 第一个进度条的是在活跃（焦点在cmder时）的窗口透明度，第二个则是在非活跃时的窗口透明度。

### 6、别名机制
cmd：修改`D:\cmder\config\user_aliases.cmd`

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/20211220142654.png width=75% />
</div>
命令方式：
```bash
alias ls='ls --color=tty'
```
bash：直接使用alias命令添加或在下面的文件中添加：

```code
$CMDER_ROOT/config/profile.d/*.sh
$CMDER_ROOT/config/user-profile.sh
$HOME/.bashrc
```
Power Shell：直接使用alias命令添加或在下面的文件中添加：

```code
'$ENV:CMDER_ROOT\config\profile.d\*.ps1'
'$ENV:CMDER_ROOT\config\user-profile.ps1'
```

## 快捷键
| 说明    | 快捷键                |
| ------- | --------------------- |
| 新建tab | Ctrl + t              |
| 关闭tab | Ctrl + w              |
| 切换Tab | Ctrl+Tab或Ctrl+1,2... |

##  参考
[1] https://www.jianshu.com/p/979db1a96f6d  

[2] https://blog.51cto.com/13853366/2352632


