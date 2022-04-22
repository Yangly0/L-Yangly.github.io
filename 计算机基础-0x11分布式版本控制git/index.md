# 分布式版本控制 Git

摘要：Git是一个开源的分布式版本控制系统，可以有效、高速地处理从很小到非常大的项目版本管理。
<!--more-->


# 分布式版本控制 Git
Git分为 工作区、暂存区、本地仓库区和远程仓库区。

- 工作区（work directory）：项目代码文件。
- 暂存区（stage）：提交代码、解决冲突的中转站。
- 本地仓库（local repository）：连接本地代码跟远程代码的枢纽，不能联网时本地代码可先提交至该处。
- 远程仓库（remote repository）:  即保存代码的服务器。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/20211220142910.png width=75% />
</div>
## 流程

1. 分支开发与测试。
```bash
$ git clone remote          # 克隆远程仓库代码
$ git checkout -b dev-ly    # 创建开发分支
$ git add files             
$ git commit -m "xxx"
```
2. 开发分支先合并主分支，且解决分支间冲突。
```bash
$ git checkout master
$ git pull
$ git checkout local
# 切换到local分支后， 就是修改代码
# 修改完了， 就正常提交代码-------git commit
# 如果有多次local分支的提交，就合并，只有一次可以不合并
$ git rebase -i HEAD~2  # 合并提交 --2表示合并两个
# 将master内容合并到local,这个过程可能需要手动解决冲突(如果进行了上一步的话,只用解决一次冲突)
$ git rebase master -->解决冲突--> git rebase --> continue
```

3. 主分支合并，且推送到远程仓库。
```bash
# 切换到master或其他目标分支
$ git checkout master
# 将开发分支合并到主分支master
$ git merge dev-ly
# 推送到远程仓库
$ git push
```

## 配置
Git的设置文件为.gitconfig，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。
```bash
# 显示当前的Git配置
$ git config --list
# 编辑Git配置文件
$ git config -e [--global]
# 设置提交代码时的用户信息
$ git config [--global] user.name "[name]"
$ git config [--global] user.email "[email address]"
#获取帮助信息
$ git help config 
```

配置SSH：本地与远程仓库连接。
```bash
$ ssh-keygen
```

1.  三次enter 进入c:/User/.ssh/文件查看id_ras,id_rsa.pub 。
2.  再进入git->Setting 新增SSH and GPG keys。

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/blog/20211220142921.png width=75% />
</div>

## 创建
```bash
# 初始化Git代码库
$ git init
# 新建一个目录，将其初始化为Git代码库
$ git init [project-name]
# 链接远程仓库（必须提前创建）
$  git remote add origin git@github.com:liuyangly1/project.git
# 下载一个项目和它的整个代码历史
$ git clone [url]
```

```bash
$ mkdir name
$ cd name
$ pwd
$ git init (.git文件)  #初始化版本库 
$ ls -ah #查看隐藏文件
#创建忽略文件
$ touch  .gitignore
$ vim .gitignore
>>> .idea/*
>>> .history/*
>>> .vscode/*
>>> .gitignore
# 关联
git remote add origin git@github.com:liuyangly1/ProgramLife.git 
git push -u origin master/main
```

## 查看
### 1、文件状态
```bash
# 显示有变更的文件
$ git status
```

### 2、版本状态
```bash
# 显示当前分支的版本历史
$ git log

# 显示commit历史，以及每次commit发生变更的文件
$ git log --stat

# 搜索提交历史，根据关键词
$ git log -S [keyword]

# 显示某个commit之后的所有变动，每个commit占据一行
$ git log [tag] HEAD --pretty=format:%s

# 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
$ git log [tag] HEAD --grep feature

# 显示某个文件的版本历史，包括文件改名
$ git log --follow [file]
$ git whatchanged [file]

# 显示指定文件相关的每一次diff
$ git log -p [file]

# 显示过去5次提交
$ git log -5 --pretty --oneline

# 显示所有提交过的用户，按提交次数排序
$ git shortlog -sn

# 显示当前分支的最近几次提交
$ git reflog
```

### 3、差异状态
```bash
# 显示暂存区和工作区的差异
$ git diff

# 显示暂存区和上一个commit的差异
$ git diff --cached [file]

# 显示工作区与当前分支最新commit之间的差异
$ git diff HEAD

# 显示两次提交之间的差异
$ git diff [first-branch]...[second-branch]

# 显示今天你写了多少行代码
$ git diff --shortstat "@{0 day ago}"
```

### 3、提交状态
```bash
# 显示指定文件是什么人在什么时间修改过
$ git blame [file]

# 显示某次提交的元数据和内容变化
$ git show [commit]

# 显示某次提交发生变化的文件
$ git show --name-only [commit]

# 显示某次提交时，某个文件的内容
$ git show [commit]:[filename]
```

## 操作
### 工作区：添加、撤销、删除、更名和恢复
#### 1、添加
```bash
# 添加指定文件到暂存区
$ git add [file1] [file2] ...

# 添加指定目录到暂存区，包括子目录
$ git add [dir]

# 添加当前目录的所有文件到暂存区, 
$ git add .

# 添加每个变化前，都会要求确认
# 对于同一个文件的多处变化，可以实现分次提交
$ git add -p
```

#### 2、撤销
```bash
# 撤销文件修改，恢复到暂存区或本地代码库（取决于文件在修改前的状态）
$ git restore  [file] 
# 文件从暂存区撤回到工作区，保留文件最后一次修改的内容
$ git restore --staged [file] 
```

#### 3、删除
```bash
# 删除工作区文件，并且将这次删除放入暂存区
$ git rm [file1] [file2] ...
```

#### 4、更名
```bash
# 改名文件，并且将这个改名放入暂存区
$ git mv [file-original] [file-renamed]
```

#### 5.恢复
```bash
# 恢复暂存区的指定文件到工作区
$ git checkout [file]

# 恢复某个commit的指定文件到暂存区和工作区
$ git checkout [commit] [file]

# 恢复暂存区的所有文件到工作区
$ git checkout .
```

### 暂存区：提交和撤销
#### 1、 提交
```bash
# 提交暂存区到仓库区
$ git commit -m [message]

# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m [message]

# 提交工作区自上次commit之后的变化，直接到仓库区
$ git commit -a

# 提交时显示所有diff信息
$ git commit -v

# 使用一次新的commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
$ git commit --amend -m [message]

# 重做上一次commit，并包括指定文件的新变化
$ git commit --amend [file1] [file2] ...
```

#### 2、 撤销
```bash
# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
$ git reset [file]

# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard

# 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
$ git reset [commit]

# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
$ git reset --hard [commit]

# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
$ git reset --keep [commit]

# 新建一个commit，用来撤销指定commit
# 后者的所有变化都将被前者抵消，并且应用到当前分支
$ git revert [commit]

# 暂时将未提交的变化移除，稍后再移入
$ git stash
$ git stash pop
```

#### 3、删除
```bash
# 不删除物理文件，仅将该文件从缓存中删除
$ git rm --cached [file]
# 不仅将该文件从缓存中删除，还会将物理文件删除（不会回收到垃圾桶）
$ git rm --f [file]
```

### 本地仓库与远程仓库区：拉取与同步
#### 1、显示
```bash
# 显示所有远程仓库
$ git remote -v

# 显示某个远程仓库的信息
$ git remote show [remote]
```

#### 2、拉取
```bash
# 下载远程仓库的所有变动，并不改变代码，工作区和缓存区。
# 后续通过git diff FETCH_HEAD 来比较，再同步git merge FETCH_HEAD。
$ git fetch [remote]  # 推荐

# 取回远程仓库的变化，并与本地分支合并，等价于git fetch + git merge。
$ git pull [remote] [branch]
```

#### 3、上传
```bash
# 上传本地指定分支到远程仓库
$ git push [remote] [branch]

# 强行推送当前分支到远程仓库，即使有冲突
$ git push [remote] --force

# 推送所有分支到远程仓库
$ git push [remote] --all
```

#### 4、连接
```bash
# 增加一个新的远程仓库，并命名
$ git remote add [shortname] [url]
```

## 分支
### 1、查看
```bash
# 列出所有本地分支
$ git branch

# 列出所有远程分支
$ git branch -r

# 列出所有本地分支和远程分支
$ git branch -a
```

### 2、创建
```bash
# 新建一个分支，但依然停留在当前分支
$ git branch [branch-name]

# 新建一个分支，并切换到该分支
$ git checkout -b [branch]

# 新建一个分支，指向指定commit
$ git branch [branch] [commit]

# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch] [remote-branch]

# 建立追踪关系，在现有分支与指定的远程分支之间
$ git branch --set-upstream [branch] [remote-branch]
```

### 3、切换
```bash
# 切换到指定分支，并更新工作区
$ git checkout [branch-name]

# 切换到上一个分支
$ git checkout -
```

### 4、合并
```bash
# merge：分支合并，则还是两个分支，只不过在merge后这个点交汇。。
git checkout main
git pull origin main
git merge dev 

# rebase：分支衍合，分支衍合不会保留合并的日志，不留痕迹，而分支合并则会保留合并的日志。
git checkout main
git rebase dev
```

注意：如果有冲突，会提示你，调用git status查看冲突文件。解决冲突，然后调用git add或git rm将解决后的文件暂存。所有冲突解决后，git commit 提交更改。

**合并问题：** `Error: Your local changes to the following files would be overwritten by merge: xxx/...`。

> 原因：当前分支有未跟踪的文件，checkout 命令会覆盖它们，请缓存(stash)或者提交(commit)。

- 选择A：未跟踪文件的内容改动很重要，保存修改。

```bash
# 两种方法：缓存(stash)或者提交(commit)。
# 方法1：把当前任务暂存起来，再获取远端的最新文件，然后合并。
$ git stash //暂存正在进行的工作
$ git pull
$ git stash pop //合并暂存的代码

# 方法2：发起一个commit存到提交历史。
$ git add .
$ git commit -m 'commit message'
```

- 选择B：未跟踪文件的内容改动不重要，放弃修改

```bash
# 两种方法：清除修改和强制分支切换

# 方法1：清除未跟踪文件【推荐做法】
$ git clean -n         # 清除文件预览
$ git clean -f         # 强制清除文件

>>> D:\Android\***\*****>git clean -n Would rempve app/build.gradle
>>> D:\Android\***\*****>git clean -f Removing app/build.gradle

# 方法2：强制分支切换，强制切换分支命令如下，结果同提示说的那样，会直接覆盖未跟踪的文件。注意：这个方式很粗暴，日常切换的时候，还是尽量不要使用-f强制切换，因为没有覆盖提示，很容易发生文件修改丢失。

$ git checkout -f <branch>    # <branch>为要切换到的分支名，注意不带“<>”
>>> Switched to branch 'master' Your branch is up to date with 'origin/master'.
```

### 5、删除
```bash
# 删除分支
$ git branch -d [branch-name]

# 删除远程分支
$ git push origin --delete [branch-name]
$ git branch -dr [remote/branch]
```

##  文件忽略
文件忽略配置：`.gitignore`。
常用模板：[gitignore](https://github.com/github/gitignore)。

```
语法：
- 空格不匹配任意文件，可作为分隔符，可用反斜杠转义
- #开头：标识注释，可以使用反斜杠进行转义
- ! 开头：标识否定，该文件将会再次被包含，如果排除了该文件的父级目录，则使用 ! 也不会再次被包含。可以使用反斜杠进行转义
- / 结束：只匹配文件夹以及在该文件夹路径下的内容，但是不匹配该文件
- / 开头：匹配文件
- 如果一个模式不包含斜杠，则它匹配相对于当前 .gitignore 文件路径的内容，如果该模式不在 .gitignore 文件中，则相对于项目根目录
- ** 匹配多级目录，可在开始，中间，结束
- ? 通用匹配单个字符
- [] 通用匹配单个字符列表
```

常用匹配示例：
```
bin/ ：忽略当前路径下的bin文件夹，该文件夹下的所有内容都会被忽略，不忽略 bin 文件
/bin ：忽略根目录下的bin文件
/*.c ：忽略 cat.c，不忽略 build/cat.c
debug/*.obj ： 忽略 debug/io.obj，不忽略 debug/common/io.obj 和 tools/debug/io.obj
**/foo ： 忽略/foo, a/foo, a/b/foo等
a/**/b ： 忽略a/b, a/x/b, a/x/y/b等
!/bin/run.sh ： 不忽略 bin 目录下的 run.sh 文件
*.log ： 忽略所有 .log 文件
config.php ： 忽略当前路径的 config.php 文件
```


## 标签
```bash
# 列出所有tag
$ git tag

# 新建一个tag在当前commit
$ git tag [tag]

# 新建一个tag在指定commit
$ git tag [tag] [commit]

# 删除本地tag
$ git tag -d [tag]

# 删除远程tag
$ git push origin :refs/tags/[tagName]

# 查看tag信息
$ git show [tag]

# 提交指定tag
$ git push [remote] [tag]

# 提交所有tag
$ git push [remote] --tags

# 新建一个分支，指向某个tag
$ git checkout -b [branch] [tag]
```

## 插件
### pre-commit
限制日志长度和提交的文件类型。
链接: [link](https://pre-commit.com/)。
1. 安装。
```bash
$ pip install pre-commit
$ pre-commit --version
pre-commit 2.7.1
```

2. 配置文件：`.pre-commit-config.yaml`。
```yaml
# See https://pre-commit.com for more information
# See https://pre-commit.com/hooks.html for more hooks
repos:
-   repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.0.1
    hooks:
    -   id: trailing-whitespace
    -   id: end-of-file-fixer
    -   id: check-docstring-first
    -   id: check-json
    -   id: check-added-large-files
    -   id: check-yaml
    -   id: debug-statements
    -   id: name-tests-test
    -   id: double-quote-string-fixer
    -   id: requirements-txt-fixer
-   repo: https://github.com/PyCQA/flake8
    rev: 4.0.1
    hooks:
    -   id: flake8
        additional_dependencies: [flake8-typing-imports==1.7.0]
-   repo: https://github.com/pre-commit/mirrors-autopep8
    rev: v1.5.7
    hooks:
    -   id: autopep8
-   repo: https://github.com/asottile/reorder_python_imports
    rev: v2.6.0
    hooks:
    -   id: reorder-python-imports
        args: [--py3-plus]
-   repo: https://github.com/asottile/pyupgrade
    rev: v2.29.1
    hooks:
    -   id: pyupgrade
        args: [--py36-plus]
-   repo: https://github.com/asottile/add-trailing-comma
    rev: v2.2.1
    hooks:
    -   id: add-trailing-comma
        args: [--py36-plus]
-   repo: https://github.com/asottile/setup-cfg-fmt
    rev: v1.20.0
    hooks:
    -   id: setup-cfg-fmt
-   repo: https://github.com/pre-commit/mirrors-mypy
    rev: v0.920
    hooks:
    -   id: mypy
        additional_dependencies: [types-all]
```

```
# 注意： .flak8常见问题
ignore = C901, W503, F405, E731, F401, F403, F841, F9
# C901: is too complex
# W503: line break before binary operator
# F405: may be undefined, or defined from star imports
exclude =
    *migrations*,
    *.pyc,
    .git,
    .cover,
    __pycache__,
    */node_modules/*,
    */templates_module*,
    */bin/*,
    packages/*
max-line-length = 120
max-complexity = 12
format = pylint
show_source = True
statistics = True
count = True
```

3. 安装pre-commit库。
```bash
$ pre-commit install
pre-commit installed at .git/hooks/pre-commit
$ pre-commit clean
```

4.  运行。
```bash
$ pre-commit run --help
# 第一次 pre-commit 运行时，将会自动下载、安装并且运行 hook
$ pre-commit run --all-files
$ pre-commit run <hook_id>
$ pre-commit autoupdate
$ git commit --no-verify
```

## 个人页面
个人页面：[Yangliuly1](https://github.com/Yangliuly1)。  
- 配置参考：[link](https://zhuanlan.zhihu.com/p/373675893)。  
- GitHub 统计卡片官方仓库：[link](https://github.com/anuraghazra/github-readme-stats)。  
- 数据牌官网：[link](https://link.zhihu.com/?target=https%3A//shields.io/)。  
- 卡片颜色 渐变色：[link](http://emojihomepage.com/)。  
- Emoji: [link](http://emojihomepage.com/)。  
- logo: [link](https://hatchful.shopify.com/zh-CN/)。  
- 访问者：[link](https://visitor-badge.glitch.me/#docs)。  

### Token
生成token，然后按以下格式更新。
```bash
$ git remote set-url origin https://<your_token>@github.com/<USERNAME>/<REPO>.git

# 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

$ git remote set-url origin https://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@github.com/yangliuly1/leetcode.git
```


## 问题
### 1、连接超时

**连接超时问题1**： `ssh: connect to host github.com port 22: Connection timed out/Resource temporarily unavailable`。

> 原因：连接超时，端口问题
> 解决：1. 查看密钥是否配置成功；2.设置配置文件，（`windows-C:/Users/root/.ssh/config和  linux-/etc/ssh/ssh_config`）；3.测试。

```bash
# 1. 查看密钥是否配置成功
$ ssh -v git@github.com # 查看问题
$ cd ~/.ssh | ls
>> id_res  id_res.pub 

# 不存在，配置你的密钥
$ ssh-keygen 邮箱[522927xx@qq.com]
$ git config --global user.name  "Your Name"
$ git config --global user.email  "email@qq.com"
$ ssh-keygen
# 添加SSH-key到github->setting->SSH-keys

# 2. 设置配置文件
$ vim linux-/etc/ssh/ssh_config
##################################################
Host github.com
User 注册github的邮箱(522927xx@11.com)
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
Port 443
##################################################
# 3. 测试
$ ssh -T git@github.com
>> Hi XXX! You've successfully authenticated, 
>> but GitHub does not provide shell access.
```

**连接超时问题2**：`fatal: unable to access 'https://github.com/powerline/fonts.git/': gnutls_handshake() failed: The TLS connection was non-properly terminated.`

```
# 重置代理
$ git config --global  --unset https.https://github.com.proxy   
$ git config --global  --unset http.https://github.com.proxy
```

### 2、大小写忽略

### 3、中文问题
```bash
$ git config core.quotepath false  --global
```

### 4、第一次历史提交查询
1. commits查询提交查看；
2. 按参数访问 地址/commits/分支?after=SHA值+次数。
```
https://github.com/lodash/lodash/commits/master?after=c2616dd4f3ab267d000a2b4f564e1c76fc8b8378+34
```

### 5、数学公式显示
github现在不支持 `$$ $$$$` 公式渲染，需要采用html格式公式，或者下载浏览器插件`MathJax Plugin for Github`。

