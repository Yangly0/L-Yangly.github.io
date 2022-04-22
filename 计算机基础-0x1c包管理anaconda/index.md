# 包管理 Anaconda

摘要：**Anaconda**，中文**大蟒蛇**，是一个开源的Python发行版本，其包含了conda、Python等180多个科学包及其依赖项。
<!--more-->

# 包管理 Anaconda
**Anaconda**，中文**大蟒蛇**，是一个开源的[Python](https://baike.baidu.com/item/Python/407313)发行版本，其包含了[conda](https://baike.baidu.com/item/conda/4500060)、Python等180多个科学包及其依赖项。

## 安装与卸载
- 安装：[link](https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/)
```bash
# Linux
$ bash Anaconda3-2019.07-Linux-x86_64.sh #yes+回车,然后重启terminal。
$ vim ~/.bashrc
# export PATH=$PATH:/home/vincent/anaconda3/bin
$ source ~/.bashrc

# Windows
Anaconda3.exe / miniconda.exe
- https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/?C=M&O=A
```
- 卸载：
```bash
# Linux
$ rm -rf anaconda
# $ rm ~/.bashrc

# Windows
$ Anaconda3/Uninstall-Anaconda3.exe
```

## 常见命令
### 1、换源
清华源采用ssl证书加密，因此只能用http不能用https，也可以下载openssl软件对ssl证书处理。删除-defaul源路径。

```bash
$ conda config --remove-key channels #恢复默认源

$ conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
$ conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
$ conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
$ conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/pkgs/free/
$ conda config --add channels https://mirrors.bfsu.edu.cn/anaconda/pkgs/main/
$ conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
$ conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
$ conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda/
$ conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/menpo/
$ conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/fastai/

# 设置搜索时显示通道地址
$ conda config --set show_channel_urls yes
```
- Windwos：修改`c:/Users/xx/.condarc`。

注意：清华源采用ssl证书加密，因此只能用http不能用https，也可以下载openssl软件对ssl证书处理，并删除-defaul源路径。

```
channels:
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
  - http://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
show_channel_urls: true
```
- Linux：`$vim ~/.condarc`

```bash
channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
show_channel_urls: true
```

### 2、环境
- 创建，激活，退出环境：
```bash
$ conda create -n environment python=3.6
$ conda activate environment
$ conda deactivate

# 自动激活
$ conda config --set auto_activate_base false  #关闭自动激活状态
$ conda config --set auto_activate_base true  #关闭自动激活状态
```
- 环境查看：
```bash
$ conda env list     #显示所有的虚拟环境
$ conda info --envs  #显示所有的虚拟环境
```

- 重命名env：
```bash
$ conda create --name newname --clone oldname      //克隆环境
$ conda remove --name oldname --all      //彻底删除旧环境
```
### 3、包
- 查询包：
```bash
$ conda list #查看安装包
$ conda list -n xxx  #指定查看xxx虚拟环境下安装包
```
- 搜索包：
```bash
$ conda search XXXX
$ anaconda search -t conda tensorflow 
$ anaconda show tensorflow
```
- 安装包：
```bash
# 本地安装包
$ conda install --use-local  ~/Downloads/a.tar.bz2
# 在线安装包
$ conda install XXXX
```
- 升级和卸载包：
```bash
$ conda update xxx   #更新xxx文件包
$ conda uninstall xxx   #卸载xxx文件包

$ conda update conda          #基本升级
$ conda update anaconda       #大的升级
$ conda update -n base conda  #update最新版本的conda
$ conda update anaconda-navigator    
#update最新版本的anaconda-navigator
```
- 清理包：
```bash
$ conda clean -p      #删除没有用的包
$ conda clean -t      #删除tar包
$ conda clean -y -all #删除所有的安装包及cache
$ conda clean -a # base环境
```
- 实例：
```bash
# 1. 启动env
$ activate env
# 2. 安装cuda和cudnn
$ conda install cudatoolkit=10.1
# 3. 安装tensorflow-gpu包
$ conda install tensorflow-gpu==1.14 （太慢）
$ conda install pytorch torchvision  cudatoolkit=10.1 -c pytorch # 或者换源
```
## 问题
1. `conda create`虚拟环境或者`conda install`出现`Segmentation fault`段错误。
   - 原因：使用conda创建虚拟环境或者下载安装某个库时，下到一半网络中断，然后再一次使用同样的命令安装或者创建时出现的错误。
   - 解决：解决方法是清除未下载完的文件。（注意，执行后刚刚下载的包会全部删除（其他的虚拟环境不受影响），得从新执行相关命令。）
```bash
$ conda clean -a # base环境
```
2. `Solving environment: failed with repodata from current_repodata.json`。
   -  原因：缓存包问题。
   -  解决：清除缓存文件。

```bash
$ conda clean -a # base环境
```
3. `Solving environment: failed with initial frozen solve. Retrying with flexible solve`。
   - 原因：通道初始化失败。
   - 解决：更新conda，设置通道为灵活。


```bash
$ conda update -n base conda
$ conda config --add channels conda-forge
$ conda config --set channel_priority flexible
```
4. `cudatoolkit`路径
```
Windows: D:/Anaconda3/pkgs/cudatoolkit-10.0.130-0/
```
5. Terminal 不显示conda环境
```bash
# PowerShell 管理员模式
$ conda init powershell
$ set-ExecutionPolicy RemoteSigned
```
