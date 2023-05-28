# Hugo 博客

摘要：从Hugo安装、Hugo发布、Hugo主题和Hugo部署四个方面展开，简述Hugo搭建博客的整个流程。
<!--more-->

# hugo  博客

<div align=center>
    <img src=https://cloud-resources-data.oss-cn-chengdu.aliyuncs.com/pics/20230528231647.jpg width=75% />
</div>

## 0x01 Hugo安装

安装教程： [gohugo](https://www.gohugo.org/)

软件包：[Releases · gohugoio/hugo](https://github.com/gohugoio/hugo/releases)

**注意：安装extended扩展版本, 0.xxx.x表示对应版本号；**

Window: 

1. 下载，软件包链接下载`hugo_extended_0.xxx.x_windows-amd64.zip`；
2. 解压并配置环境，添加`hugo.exe`路径到环境路径`PATH`；
3. 测试，`$ hugo version`；

Linux:

1. 下载，`$ wget hugo_extended_0.xxx.x_linux-amd64.deb`；
2. 解压并安装，`$ sudo dpkg -i hugo_extended_0.xxx.x_linux-amd64.deb`；
3. 测试，`$ hugo version`；

## 0x02 Hugo发布

```bash
# 创建博客站点
$ hugo new site blog
$ cd blog
# 创建文章
$ hugo new posts/first_post.md
# 发布
hugo server --disableFastRender --navigateToChanged --environment pruction --bind 0.0.0.0 
```

## 0x03 Hugo主题

主题库：[themes](https://themes.gohugo.io/)

1. FixIt主题下载。

```bash
$ cd blog  # 到站点文件夹
$ git init
$ git submodule add https://github.com/hugo-fixit/FixIt.git themes/FixIt
```

2. FixIt主题配置：配置config.toml，详细参考：[入门篇 完整配置 - FixIt (lruihao.cn) ](https://fixit.lruihao.cn/zh-cn/documentation/basics/#完整配置)；
   1. 全局配置；
   2. 菜单配置；
   3. 页面配置；
   4. …

**注意：访问 http://localhost:1313/ 。**

## 0x04 Hugo部署

思路：Github部署 + Hugo页面，即每次提交博客到仓库blog（Hugo搭建而成的仓库），blog仓库会同步信息到个人页面仓库`xxx.github.io`，并发布网页 ，参考链接：[link](https://gohugo.io/hosting-and-deployment/hosting-on-github/#build-hugo-with-github-action)。

**部署流程：**

1. 创建`blog`仓库，同步blog站点；

   ```bash
   $ cd blog  # 到站点文件夹
   $ git init
   $ git add .
   $ git branch -M main
   $ git remote add origin https://github.com/xxx/blog.git
   $ git push -u origin main
   ```

2. 创建个人页面`xxx.github.io`仓库，用于构建github pages；

3. 配置ssh-key：

   a. 本地创建密钥dp_deploy_key；

   ```bash
   $ ssh-keygen -t rsa -C "xxx@xxx.com" -f ~/.ssh/dp_deploy_key  
   ```

   b. `xxx.github.io`仓库->settings -> Deploy keys -> ACTIONS_DEPLOY_KEY -> Public Key -> Allow write access，创建令牌GP_DEPLOY_KEY_PUB，并填写以下内容;

   ```bash
   $ cat ~/.ssh/dp_deploy_key.pub 
   ```

   c. `blog`仓库 -> settings -> Secrets -> ACTIONS_DEPLOY_KEY -> Private Key，创建令牌GP_DEPLOY_KEY，并填写以下内容;

   ```bash
   $ cat ~/.ssh/dp_deploy_key 
   ```

   d. 个人账号->settings->SSH andGPG keys->SSH keys，增加一个SSH keys，并填写以下内容;

   ```bash
   $ cat~/.ssh/dp_deploy_key.pub
   ```

4. 增加博客同步Action：`.github/workflows/deploy-hugo.yml`，**需要修改`external_repository`**;

   ```yaml
   name: Deploy hugo
   
   on:
     push:
       branches:
         - main  # Set a branch to deploy
     pull_request:
   
   jobs:
     deploy:
       runs-on: ubuntu-latest
       concurrency:
         group: ${{ github.workflow }}-${{ github.ref }}
       steps:
         - uses: actions/checkout@v3
           with:
             submodules: true  
             fetch-depth: 0 
   
         - name: Setup Hugo
           uses: peaceiris/actions-hugo@v2
           with:
             hugo-version: 'latest'
             extended: true
   
         - name: Build
           run: hugo --minify
   
         - name: Deploy
           uses: peaceiris/actions-gh-pages@v3
           with:
             deploy_key: ${{ secrets.GP_DEPLOY_KEY }}
             external_repository: xxx/xxx.github.io
             publish_branch: main
             publish_dir: ./public
             commit_message: ${{ github.event.head_commit.message }}
   ```

5. 增加主题更新Action：`.github/workflows/update-theme.yml`；

   ```yaml
   name: Update theme
   
   # Controls when the workflow will run
   on:
     schedule:
       # Update theme automatically everyday at 00:00 UTC
       - cron: "0 0 * * *"
     # Allows you to run this workflow manually from the Actions tab
     workflow_dispatch:
   
   jobs:
     Update-FixIt:
       runs-on: ubuntu-latest
       permissions: write-all
       steps:
         - name: Check out repository code
           uses: actions/checkout@v3
           with:
             submodules: true
             fetch-depth: 0
   
         - name: Update theme
           run: git submodule update --remote --merge themes/FixIt
         
         - name: Change remote origin
           run: |
             git remote remove origin
             git remote add origin https://github.com/L-Yangly/hugo-blog.git
   
         - name: Commit changes
           uses: stefanzweifel/git-auto-commit-action@v4
           with:
             branch: main
             commit_message: ':arrow_up: Chore(theme): update FixIt version'
             commit_author: 'github-actions[bot] <github-actions[bot]@users.noreply.github.com>'
             push_options: '--set-upstream'
   ```

## 0x05 Hugo内容

参考：[所有内容管理 - FixIt (lruihao.cn)](https://fixit.lruihao.cn/zh-cn/documentation/content-management/)

###  A. 前置参数

**Hugo** 允许你在文章内容前面添加 `yaml`, `toml` 或者 `json` 格式的前置参数。

```yaml
---
title: "我的第一篇文章"
subtitle: ""
date: 2020-03-04T15:58:26+08:00
draft: true
...
---
```

**内容加密**：FixIt 主题提供了两个前置参数用于全文加密。

- **password**: *[必需]* 加密页面内容的密码；
- **message**: *[可选]* 加密提示信息；

### B. 内容摘要

默认情况下，Hugo 自动将内容的前 70 个单词作为摘要。你可以通过在网站配置中设置 `summaryLength` 来自定义摘要长度。或者在文章开头添加 `<!--more-->` 摘要分割符，将摘要分隔符之前的内容保留为空，然后 **FixIt** 主题会将你的文章描述作为摘要。

### C. Shortcodes

为了避免这种限制，Hugo 创建了 shortcodes。 shortcode 是一个简单代码段，可以生成合理的 HTML 代码，并且符合 Markdown 的设计哲学。

内置 Shortcodes：

- figure
- gist
- highlight
- param
- ref和relref
- tweet
- vimeo
- youtube

扩展 Shortcode：

- typeit
- bilibili
- music
- mapbox
- echarts
- mermaid



---

> 作者: [L-Yangly](https://github.com/L-Yangly)  
> URL: https://L-Yangly.github.io/hugo/  

