# 交互式脚本 Zsh

摘要：Zsh（Z-shell）是一款用于交互式使用的shell，也可以作为脚本解释器来使用。
<!--more-->

# 交互式脚本 Zsh
## zsh安装与配置
```bash
$ sudo apt-get install zsh
$ which zsh
/usr/bin/zsh

# echo $SHELL查看当前默认的Shell
$ chsh -s /bin/zsh
$ chsh -s /bin/bash
```

## zsh个性化
### 1、主题Oh My Zsh
自动安装：
```bash
$ sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

$ sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

手动安装：
```bash
$ git clone https://github.com/ohmyzsh/ohmyzsh.git ~/.oh-my-zsh
# 备份
$ cp ~/.zshrc ~/.zshrc.orig
$ cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
# $ cd oh-my-zsh/tools
# $ ./install.sh

$ chsh -s $(which zsh)
```

主题位置`~/.oh-my-zsh/themes`。
```bash
$ vim ~/.zshrc
ZSH_THEME="robbyrussell"  # agnoster
$ source ~/.zshrc
```

### 2、Oh My Zsh 插件
- incr自动补全插件
```bash
$ cd ~/.oh-my-zsh/plugins/
$ mkdir incr && cd incr
$ wget http://mimosa-pudica.net/src/incr-0.2.zsh
$ vi ~/.zshrc
# 末尾
source ~/.oh-my-zsh/plugins/incr/incr*.zsh
$ source ~/.zshrc
```
- `git`：默认，提供了大量 git 的alias。
- `extrac`t：解压插件。
- `z`：`~/.oh-my-zsh/plugins`默认安装，强大的目录自动跳转命令，会记忆你曾经进入过的目录，用模糊匹配快速进入你想要的目录。`urce ~/.zshrc`。
- `autojump`：自动跳转插件。
```bash
$ apt install autojump
$ vi ~/.zshrc
# 末尾添加
. /usr/share/autojump/autojump.sh
source ~/.zshrc
```

- `zsh-autosuggestions` ：语法历史记录。
```bash
$ git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions

$ vim ~/.zshrc
# 添加或修改
plugins=(git zsh-autosuggestions)
$ source ~/.zshrc
```

- `zsh-syntax-highlighting`：语法高亮。
```bash
$ git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

$ vim ~/.zshrc
# 添加或修改
plugins=(z git extract zsh-syntax-highlighting)
$ so

## 参考
- [Ricsy-Ubuntu | 安装oh-my-zsh-简书](https://www.jianshu.com/p/ba782b57ae96)
```

### 3、Oh My Zsh更新与卸载
```bash
# 设置更新时间
$ vi ~/.zshrc
export UPDATE_ZSH_DAYS=13
# 禁用自动更新
vi ~/.zshrc
DISABLE_AUTO_UPDATE="true"
# 更新
$ upgrade_oh_my_zsh
# 卸载
$ uninstall_oh_my_zsh zsh
```

## zsh卸载
```bash
$ apt-get uninstall zsh
$ chsh -s /bin/bash
```


