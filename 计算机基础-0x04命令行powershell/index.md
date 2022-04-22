# 命令行 PowerShell

摘要：Windows PowerShell 是一种命令行外壳程序和脚本环境，使命令行用户和脚本编写者可以利用 .NET Framework的强大功能。
<!--more-->

# Title


# 命令行 PowerShell

##  Oh My Posh 插件

### 1. 安装

```bash
$ $Install-Module posh-git -Scope CurrentUser
$ Install-Module oh-my-posh -Scope CurrentUser
 
$ Set-Prompt
$ Set-Theme Paradox
# 主题根据自己喜好来选，可以去上面 GitHub 链接向下翻查看预览
```

### 2. 配置

打开默认 PowerShell 的启动配置文件（如果没有就创建）

```bash
# 如果之前没有配置文件，就新建一个 PowerShell 配置文件
$ if (!(Test-Path -Path $PROFILE )) { New-Item -Type File -Path $PROFILE -Force }

# 用记事本打开配置文件
$ notepad $PROFILE

# 系统上禁止运行脚本

$ set-executionpolicy remotesigned
```

```bash
# 弹出的窗口中粘贴以下内容：
$ Import-Module posh-git 
$ Import-Module oh-my-posh 
# 设置主题Paradox/Sorin/Agnoster/Avit/robbyrussell
$ Set-Theme Sorin
```

## Oh My Zsh 插件

有像高亮、自动补全等提高效率和易用性的插件。Oh My Posh 原生就支持高亮了，自动补全也有专门插件实现，那就是 PSReadline , 将官方给的 [样例](https://raw.githubusercontent.com/PowerShell/PSReadLine/master/PSReadLine/SamplePSReadLineProfile.ps1) 全部复制进去，注意命名空间的申明（using namespace）一定要在最上端。并在 Import-Module PSReadLine 下面加上一行：

```bash
# 注: 已经自动安装了
$ notepad $PROFILE
# 用于实现 Tab 触发选择模式。
$ Set-PSReadlineKeyHandler -Key Tab -Function MenuComplete
```

## scoop 插件

- link: [scoop](https://github.com/lukesampson/scoop)

### 1. 环境配置与安装

1. [PowerShell 5](https://aka.ms/wmf5download) (or later, include  PowerShell Core) and [.NET Framework 4.5](https://www.microsoft.com/net/download) (or later)

```bash
$ $PSVersionTable.PSVersion.Major   #查看Powershell版本
$ $PSVersionTable.CLRVersion.Major  #查看.NET Framework版本
```

2. 允许本地脚本的执行

```bash
$ Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser # 选择A[全是]
$ Get-ExecutionPolicy
```

3. 安装

- 默认目录C:\Users\user(自己的用户名)\scoop

```bash
$ $env:SCOOP='D:\Scoop'
$ [Environment]::SetEnvironmentVariable('SCOOP', $env:SCOOP, 'User')
$ iex (new-object net.webclient).downloadstring('https://get.scoop.sh')
```

4. 问题：无法链接即FullyQualifiedErrorId : WebException。或者操作超时。

```bash
# 注意删除对应目录下scoop文件夹
$wc = new-object net.webclient
$wc.Proxy.Credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials
iex (new-object net.webclient).downloadstring('https://raw.githubusercontent.com/lukesampson/scoop/master/bin/install.ps1')
```

- 手动复制文件https://raw.githubusercontent.com/lukesampson/scoop/master/bin/install.ps1写入到`1.ps1`文件

```
./1.ps1
```

- 然后定位到`Downloading scoop...`，手动下载，复制到`$dir`。

```bash
$ mkdir -r D:\Scoop\apps\scoop\current
# D:\Scoop\apps\scoop\current\scoop.zip
https://github.com/lukesampson/scoop/archive/master.zip

$ mkdir -r D:\Scoop\buckets\main
# D:\Scoop\buckets\main\main-bucket.zip
https://github.com/ScoopInstaller/Main/archive/master.zip
```

- 然后注释掉以下代码，继续运行`1.ps1`。

```
# 注释掉文件夹检测
#if (installed 'scoop') {
#    write-host "Scoop is already installed. Run 'scoop update' to get the latest version." -f red
#    # don't abort if invoked with iex that would close the PS session
#    if ($myinvocation.mycommand.commandtype -eq 'Script') { return } else { exit 1 }
#}

注释掉两个dl $zipurl $zipfile
# dl $zipurl $zipfile

#dl $zipurl $zipfile
```

### 2.  scoop 命令

```bash
$ scoop help
$ scoop search git
$ scoop install git
$ scoop uninstall git 

$ scoop update #更新scoop
$ scoop update 7zip #更新7zip
$ scoop * #更新全部
```

### 3. scoop 加速下载

```bash
$ scoop install aria2

$ scoop config aria2-max-connection-per-server 16
$ scoop config aria2-split 16
$ scoop config aria2-min-split-size 1M
```

### 4. scoop bucker

- [scoop-directory](https://github.com/rasa/scoop-directory/blob/master/by-apps.md)

```bash
$ scoop bucket known # 查看安装了那些库

# scoop bucket add extras https://github.com/lukesampson/scoop-extras.git
$ scoop bucket add extras
# scoop install sudo git curl 7zip zotero

$ scoop bucket add dorado https://github.com/chawyehsu/dorado
# 或者使用国内镜像，速度快但是非实时同步
$ scoop bucket add dorado https://gitee.com/h404bi/dorado.git

$ scoop install dorado/<软件名>
```

### 5. 配置导出

```bash
$ scoop list > D:/scoop/ScoopAppList.txt
```

## Gsodo 插件

- https://github.com/gerardog/gsudo

```bash
$ scoop install gsudo
$ sudo -v
```

## Git 插件

步骤：git安装->环境配置,path->`D:\Git\bin`。

## 字体渲染

1. 下载更纱黑体 [GitHub 页面](https://github.com/be5invis/Sarasa-Gothic/releases)  ttf格式下载，并选中所有字体文件，右键选择安装。或者下载link: [Sarasa-Gothic](https://github.com/be5invis/Sarasa-Gothic/releases)，解压到复制到 `C:\Windows\Font\`目录。
2. 配置：鼠标右键->属性->字体->等距更纱黑体SC。
3. 等宽字体有「等距更纱黑体 SC」或者「Sarasa Mono SC Regular」。
4. 在 Windows Terminal 的「Settings - profiles - defaults」中加入"fontFace": "等距更纱黑体 SC"。

## Alias 别名

```bash
# 查看
$ gal
$ Get-Alias
# 创建
$ New-Alias list get-childitem
# 设置
$ Set-Alias  list get-childitem
# 删除
$ Remove-Item ALIAS:\list
# Set-Alias或New-Alias命令创建的别名在关闭此Session后即会失效

# 永久创建
$ PROFILE
# 不存在创建
$ New-Item -Type file -Force $PROFILE
# 打开文件，键入内容Get-ChildItem -Name创建别名ls：
$ notepad $PROFILE
#####################################
function LoginTencent {
    ssh 用户名@ip地址 -p ssh的端口号
}
set-alias logten LoginTencent
#######################################
```

## PowerShell 问题

### 1. 尝试新的跨平台 PowerShell `https://aka.ms/pscore6`

1. 前往[Githubv7.0.0 Release of PowerShell](https://github.com/PowerShell/PowerShell/releases)下载对应的msi文件。

```
Releases->Assets->PowerShell-7.0.2-win-x64.msi
```

### 2. ARNING: Terminal type 'xterm-256color' is not supported, 'xterm' will be used as default.

```bash
# 查看xterm终端支持的颜色
$ tput colors
# 查看终端类型
$ echo $TERM
# 修改1
$ vim ~/.bashrc
####################################
if [ "$TERM" == "xterm" ]; then
export TERM=xterm-256color
fi
####################################
# 修改2
$ vim ~/.Xresources
####################################
xterm*termName: xterm-256color
####################################
```

### 3. Powershell不显示conda环境

- PowerShell 管理员模式

```bash
$ conda init powershell
$ set-ExecutionPolicy RemoteSigned
```


