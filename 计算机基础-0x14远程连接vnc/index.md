# 远程连接 VNC Connect

摘要：VNC (Virtual Network Console)是虚拟网络控制台的缩写。它 是一款优秀的远程控制工具软件，由著名的 AT&T 的欧洲研究实验室开发的。
<!--more-->

# 远程连接 VNC Connect
## 服务端 VNC® Server
### Windows
#### 1、安装
地址：[link](https://www.realvnc.com/en/connect/download/viewer/)  
安装并激活：
```
# 激活码
BQ24G-PDXE4-KKKRS-WBHZE-F5RCA  
BQ24G-PDXE4-KKKRS-WBHZE-F5RCA  
8ZEZH-QPANM-NX3A5-8C4TS-8B97A
```

####  2、配置
1. 右上角->Options。
2. Security->``( 电脑开机密码)。
3. `Users&Permissions`：`Control desktop using keyboard and Control desktop using mouse`。

### Linux
Ubuntu 存储库中还有几种不同的 VNC 服务器，如 [TightVNC](https://www.tightvnc.com/) ， [TigerVNC](http://tigervnc.org/) 和 [x11vnc](http://www.karlrunge.com/x11vnc/) 。每个 VNC 服务器在速度和安全性方面都有不同的优点和缺点。

#### 1、安装
```bash
# 离线安装
$ wget https://www.realvnc.com/download/file/viewer.files/VNC-Viewer-6.21.1109-Linux-x64.deb
$ dpkg -i VNC-Viewer-6.21.1109-Linux-x64.deb

# 在线安装
$ sudo apt-get install vnc4server
```

#### 2、配置
1. 编辑配置文件。复制最后两行并去掉行首注释符并"修改"。
```bash
# $ vim /etc/sysconfig/vncservers
# VNCSERVERS="2:myusername"
# VNCSERVERARGS[2]="-geometry 800x600 -nolisten tcp -nohttpd -localhost"

# VNCSERVERS="1:root"  
# VNCSERVERARGS[1]="-geometry 1024x768"

# 配置说明：
# 1. VNCSERVERS：配置登录远程桌面的用户名。
# 2. VNC的默认监听端口是5900，监听端口规则为590+usernumber，
#    如 2:root对应端口号5902。
# 3. VNCSERVERARGS[2] 登录桌面配置：
#    2 为用户序号
#    1366x768 为分辨率
#    -nolisten tcp 为阻止tcp包
#    -nohttpd 为阻止http包
#    -localhost 代表只监听本地
```

2. 配置linux桌面显示程序 [link](https://blog.csdn.net/qq_41204464/article/details/103918743)。
```bash
# 正常远程显示ubuntu的桌面
$ sudo apt-get install ubuntu-desktop gnome-panel gnome-terminal gnome-settings-daemon metacity nautilus 

# 初次启动的时候需要设置初始密码,且root根目录下面生成配置文件/root/.vnc/xstartup
$ vncserver  -geometry 1900x1000 :1
$ vnc4server -kill :1
$ vim /root/.vnc/xstartup

# 取消注释
# unset SESSION_MANAGER                                                                                       # exec /etc/X11/xinit/xinitrc 

# 注释
# xterm -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" twm &

# 添加
# gnome-terminal&             
# gnome-panel&                

# gnome-session &             
# gnome-settings-daemon &     
# metacity &
# nautilus &
```

3. 改变xstartup的权限
```bash
$ chmod 777 ~root~/.vnc/xstartup
```

4. 防火墙
```bash
$ vim /etc/sysconfig/iptables
# -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
# ADD
# -A INPUT -m state --state NEW -m tcp -p tcp --dport 5901 -j ACCEPT

# 重启
$ /etc/init.d/iptables restart
```

5. 添加额外包75dpi，注意查看日志`~/.vnc/xxx:1.log`。
```bash
$ sudo apt-get install xvfb xfonts-100dpi \
	xfonts-75dpi xfonts-cyrillic xorg dbus-x11
```

#### 3、操作：查询、开启、关闭、重启、重置密码和分辨率

```bash
# 查询
$ ps -ef|grep -i vnc
# 开启
vncserver  -geometry 1900x1000 :1
# 关闭
$ vncserver -kill :1 （推荐，注意空格）

# 重置密码, 两次输入即可
$ vncpasswd

# 分辨率
$ vncserver -dpi 100 -geometry 1900x1000 ：2
$ vncserver -geometry 1900x1000 :1 
$ vim /etc/sysconfig/vncservers
# "-geometry 800x600" # 文件修改
```

### 常见问题
1. 打开终端。
```
任意文件夹 -> 右键 -> 选择终端
```

2. 警告问题：`warning: user: 2 is taken because of /tmp/.X2-lock remove this file if there is no x server user: 2.`。
```bash
$ rm -rf  /tmp/.X2-lock
```

3. VNC字体放大。
```
终端 -> 工具栏 -> 查看 -> 放大
```

4. 若Linux开启了防火墙，就需要手工开启相应的端口。
```bash
$ iptables -I INPUT -p tcp --dport 5902 -j ACCEPT
```

5. VNC浏览器输入中文。
```
linux 系统语言设置
```

6. VNC服务端与客服端内容通讯（复制和剪贴）。
```
保持 $vncconfig 命令打开模式，该命令弹出的窗口有三个选项，默认一般都是全部勾上的。
注意：命令窗口一直开着才能通讯。
```

7. 连接拒绝：`The connection was refused by the host computer service vncserver restart`。
```bash
# 关闭vnc
$ vncserver -kill  :id 
# 重启vnc
$ vncserver -geometry 1900x1000 :1 
# 出现如下提示，删除即可。
# warning: user: 2 is taken because of /tmp/.X2-lock
# remove this file if there is no x server user: 2
$ rm -rf  /tmp/.X2-lock
```

8. 启动后出现灰色背景+命令行：anaconda 路径问题。
```bash
$ vncserver -kill :id
# 临时修改路径
$ echo $PATH
#/media/d/liuyang/softwares/anaconda3/bin:/media/d/liuyang/softwares/anaconda3/condabin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/local/cuda/bin
$ export PATH = /media/d/liuyang/softwares/anaconda3/condabin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/local/cuda/bin

$ vncserver -geometry 1900x1000 :1 
```

## 客户端 VNC® Viewer

### Windows
1. 下载安装VNC
2. 打开VNC，输入ip+VNCport，点击Connect，输入预设密码.
