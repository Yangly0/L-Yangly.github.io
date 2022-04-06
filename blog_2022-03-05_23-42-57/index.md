# Hugo搭建博客


摘要：从Hugo安装、Hugo主题、Hugo博客和Hugo部署四个方面展开，简述Hugo搭建博客的整个流程。

<!--more-->

# Hugo搭建博客

## Hugo 安装

资源链接：[gohugoio](https://github.com/gohugoio/hugo) 和 [gohugo](https://www.gohugo.org/)。

1. window下离线下载`hugo.exe`软件。

`https://github.com/gohugoio/hugo/releases/hugo_extended_0.96.0_Windows-64bit.zip`

2. 环境配置，添加`hugo.exe`路径到`PATH`。
3. CMD测试，`$hugo --version`。

## Hugo 主题

主题库：[themes](https://themes.gohugo.io/)。

1. 主题下载。

```bash
$ git init
$ git submodule add https://github.com/HEIGE-PCloud/DoIt.git themes/DoIt
```

2. 主题配置：配置TOML，[doit](https://hugodoit.pages.dev/zh-cn/)。

## Hugo 博客

```bash
# 创建博客站点
$ hugo new site blog
$ cd blog
# 创建文章
$ hugo new posts/first_post.md
# 临时
$ hugo server -D 
# 构建网站
$ hugo
```

## Hugo 部署

参考链接：[link](https://gohugo.io/hosting-and-deployment/hosting-on-github/#build-hugo-with-github-action)。



Github部署 + Hugo页面：

1. 创建`blog`仓库，同步blog站点。

```bash
cd blog  # 到站点文件夹
git init
git add .
git branch -M main
git remote add origin https://github.com/xxx/blog.git
git push -u origin main
```

2. 配置`hugo-action.yml`，参考[link](https://github.com/peaceiris/actions-gh-pages)，注意`external_repository`值。
3. 创建`xxx.github.io`仓库，构建github pages。

4. 配置ssh-key，`xxx.github.io`仓库->settings -> Deploy keys -> ACTIONS_DEPLOY_KEY -> Public Key -> Allow write access，`blog`仓库 -> settings -> Secrets -> ACTIONS_DEPLOY_KEY -> Private Key。

```bash
$ ssh-keygen -t rsa -C "xxx@xxx.com" -f ~\.ssh\id_rsa_hugo_deploy
```

