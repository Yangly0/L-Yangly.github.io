# 子系统 Windows WSL

摘要：Windows Subsystem for Linux（简称WSL）是一个在Windows 10上能够运行原生Linux二进制可执行文件（ELF格式）的兼容层。它是由微软与Canonical公司合作开发，其目标是使纯正的Ubuntu、Debian等映像能下载和解压到用户的本地计算机，并且映像内的工具和实用工具能在此子系统上原生运行。
<!--more-->

# 子系统 Windows WSL

## WSL
### 1、安装
配置:
1. 开始->设置->更新和安全->开发者选择->开发者模式
2. `win+X`->应用和功能->相关设置->启动或关闭windows功能->适用于Linux的windows子系统

安装:
1. 应用商店安装，搜索ubuntu，点击安装->设置账号和密码
2. `win+R`->`$ lxrun /install /y`->设置账号和密码
3. 安装目录`C:\Users\root\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu18.04onWindows_79rhkp1fndgsc`

### 2、换源
添加源内容：
```bash
$ sudo vim /etc/apt/sources.list

deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```

更新源内容：
```bash
$ sudo apt-get update
$ sudo apt-get upgrade
```

## WLS2
### 1、Windows版本与更新

Windwos版本 2004：[link](http://b1.download.windowsupdate.com/d/upgr/2020/02/19041.84.200218-1143.vb_release_svc_refresh_clientconsumer_ret_x64fre_zh-cn_4755f5fabf915c618f41bfc5f280069373f5bc4f.esd)。
```bash
# 1. cmd ->命令如下
>>> Get-ComputerInfo | select WindowsProductName, WindowsVersion, OsHardwareAbstractionLayer

WindowsProductName WindowsVersion OsHardwareAbstractionLayer
------------------ -------------- --------------------------
Windows 10 Pro     2004           10.0.19041.1

# 2. win+r -> winver
```

Windows 版本更新
1. Win->设置->更新和安全->Windows预览体验计划 或 微软 Windows 10 易升。
2. 如果此版本尚不受支持，可以手动更新。先找到安装文件，解析安装文件，手动安装，即`C:\Windows10Upgrade\Configuration.ini`->esd格式文件->格式转换为iso(DISM++)->解压->setup.

### 2、配置

1. 管理员权限运行Windows PowerShell

```bash
# 开启Windows Subsystem for Linux特性
#dism.exe /online /enable-feature/featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
# 开启Virtual Machine Platform特性
#dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# 启用虚拟机平台
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
# 启用Linux子系统功能
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```

2. 命令执行完之后需要重启，根据情况选择自动或手动重启

```bash
>>> restart-computer
```

### 3、安装
1. 应用商店搜索并安装linux发行版本，以ubuntu为例，安装完成后输入账号和密码初始化登录。注意，以前安装过wsl，后来卸载了，可以点更新，重新安装。
2. 确认wsl版本。

```bash
wsl --list --verbose
```

3. 需要先更新WSL 2 Linux 内核：[link](https://docs.microsoft.com/zh-cn/windows/wsl/wsl2-kernel)。

```
# 下载
https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi
# 以批准此安装
```

4. 设置WSL 2为默认版本。

```bash
$ wsl --set-version Ubuntu-18.04 2 #Ubuntu-18.04为我的Linux版本。
$ wsl --set-default-version 2 ##该命令命令可以在以后安装 Linux 的时候默认启用 WSL2。
```

## WSL 命令
1. 启动与关闭。
```bash
#启动默认的WSL2 和 Linux
$ wsl
$ wsl -d Ubuntu-18.4 #指定版本

# 关闭
$ wst -t [name]
#关闭所有正在运行的 Linux 和 WSL 2
$ wsl --shutdown
```

2. 系统和版本查看。
```bash
$ wsl -l 
$ wsl -l -v
```

3. 内存。
```bash
$ wsl free -m
```

4. 卸载。
```bash
# 显示出你安装的列表。
$ wslconfig /l  
# #Ubuntu-18.04为上述列表中的名字   注销子系统
$ wslconfig /u Ubuntu-18.04
# 移除目录
$ get-appxpackage CanonicalGroupLimited.Ubuntu18.04onWindows | remove-Appxpackage

# 删除对应文件夹
$ cd C:\Users\root\AppData\Local\Packages\
$ rm -rf CanonicalGroupLimited.Ubuntu18.04onWindows_79rhkp1fndgsc
```

## LxRunOffline

### 1、安装

link：[LxRunOffline](https://github.com/DDoSolitary/LxRunOffline/releases)，下载即可使用LxRunOffline.exe。

环境配置：我的电脑->右键属性->高级系统设置->环境变量->系统变量Path->添加`../LxRunOffline.exe`。

### 2、常用命令

```bash
//已经安装的WSL
$ LxRunOffline list 
//还原WSL
$ LxRunOffline install -n <wsl_name> -d <res_path> -f <back_path>
//卸载WSL
$ LxRunOffline uninstall -n <wsl_name>
//备份WSL
$ LxRunOffline export -n <wsl_name> -f <back_path>
//启动一个WSL
$ LxRunOffline run -n <wslname>
```

### 3、迁移demo

**注意，对于WSL2，千万不能点击ex53.vhdx文件，否则无法访问。**
```bash
# 查看系统
$ lxrunoffline list
# 先创建一个文件夹D:\Ubuntu，再移动到D:\Ubuntu
$ mkdir D:\Ubuntu
$ lxrunoffline move -n Ubuntu-18.04 -d D:\Ubuntu
# 时间比较久。完成后，输入下面命令查看
$ lxrunoffline get-dir -n Ubuntu-18.04
# 若出现unregister错误只需用LxRunOffline重新注册一下即可
$ lxrunoffline register -n Ubuntu-18.04 -d D:\Ubuntu
```

## CUDA on WSL
### 1、CUDA windows驱动

- https://docs.nvidia.com/cuda/wsl-user-guide/index.html
- https://blog.csdn.net/xautzxc/article/details/107610353?utm_medium=distribute.pc_relevant.none-task-blog-title-8&spm=1001.2101.3001.4242

- windows安装驱动： https://developer.nvidia.com/cuda/wsl/download

### 2、WSL安装cuda toolkit 10.1

- [WSL2 版本](https://developer.nvidia.com/cuda-toolkit)

1. 安装，不需要安装nvidia-driver, 注意版本。

```bash
# https://developer.nvidia.com/cuda-toolkit
$ wget https://developer.download.nvidia.com/compute/cuda/11.1.0/local_installers/cuda_11.1.0_455.23.05_linux.run
$ sudo bash cuda_11.1.0_455.23.05_linux.run --no-opengl-libs
# Driver: Not Selected。  X表示选中
```

2. 环境配置。

```bash
$ export PATH=/usr/local/cuda-11.1/bin${PATH:+:${PATH}}
$ export LD_LIBRARY_PATH=/usr/local/cuda-11.1/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

3. 测试。

```bash
$ cd ~/NVIDIA_CUDA-10.1_Samples/1_Utilities/deviceQuery
make
$ ./deviceQuery
```

4. 卸载。

```bash
$ cd /usr/local/cuda/bin
$ ./cuda-uninstaller
Successfully uninstalled
```
4. Miniconda安装

##  问题
### 1. `WslRegisterDistribution failed with error: 0x800701bc`

- 官方的讨论连接https://github.com/microsoft/WSL/issues/5393
- 安装WSL2内核https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi
- https://blog.csdn.net/huiruwei1020/article/details/107551106


