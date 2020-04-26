---
title: Mac 上配置 Hexo
date: 2020-04-26 17:10:50
categories:
- Web前端
tags:
- Mac
- hexo
---

时隔许久又开始配置 Hexo 了。。。上次还是在上学的时候配置的，直接更新怕不兼容，干脆重新配一遍，正好也把电脑里的笔记什么的都整理一下，顺便把配 Hexo 时踩的坑记一下。

<!--more-->

## Git pages
Git pages 就比较简单了，在 github 上新建一个仓库，取名 username.github.io，其中 "username" 就是你的 github 登录名，创建仓库时要选公开的，私有的不行。（以前版本的 git pages 是在仓库的 “gh-pages” 分支上作为网页的，现在改成 master 分支了）

## Hexo 配置
### 安装 Hexo
首先安装 Hexo，按照 [Hexo 官网](https://hexo.io/zh-cn/docs/) 的方法：
```bash
npm install hexo-cli -g

```
还要安装一个用于部署到 github 上的插件：
```bash
npm install --save hexo-deployer-git
```
安装完成后创建一个 hexo 项目：
```bash
hexo init hexo
```

等待创建完成，就可以用`hexo server`或者`hexo s`启动 hexo 服务来预览了。

### 配置主题
我这里挑了个比较简约的主题 [NexT](https://github.com/theme-next/hexo-theme-next) 来替代默认的主题，如果你不需要换主题可以直接跳过这一节。

进入项目根目录，执行以下命令安装 NexT 主题：
```bash
git clone https://github.com/theme-next/hexo-theme-next themes/next
```
clone 好之后，直接删了 `themes/next/.git` 文件夹，因为以后这个项目也是要用 git 托管的，我对 next 主题的更新又不怎么在意，直接删了了事。

然后把根目录下的 `/_config.yml` 配置文件中 `theme: landscape` 改成 `theme: next` 应用上新主题。

然后是`/themes/next/_config.yml`文件，这里面的配置是针对主题的，具体配置参照[官方文档](https://theme-next.org/docs/getting-started/)。
其中的“schema”字段，是 next 主题内置的几种风格：
```yml
# Schemes
# scheme: Muse
#scheme: Mist
# scheme: Pisces
scheme: Gemini
```
想用哪个风格就放开对应的注释就好。

### 配置 Hexo
Hexo 配置参考 [官方文档](https://hexo.io/zh-cn/docs/configuration) 就好，这里记录几个比较重要的地方。

首先是 url 部分，这里用的是 git pages 方式，所以直接按照下面的方式写就好：
```yml
url: https://username.github.io/
root: /
```

"post_asset_folder: true" 可以让你用 `hexo new xxx` 命令生成的新博客文件同时生成一个同名文件夹，将这个 md 文件需要用到的图片资源放到这个同名文件夹中更好管理。然后按照[文档](https://hexo.io/zh-cn/docs/asset-folders)所述，md 文件里用到图片资源时应使用：
```
{% asset_img example.jpg This is an example image %}
```
来引用对应资源，它的规则是：
```
{% asset_path slug %}
{% asset_img slug [title] %}
{% asset_link slug [title] %}
```

然后是`deploy`部分：
```yml
deploy:
  type: git
  repo: git@github.com:username/username.github.io.git
  branch: master
  user: nickname
  email: email@gmail.com
```
"repo"部分，你也可以写成 `https://github.com/username/username.github.io.git`，区别就是 https 方式在部署的时候要手动输入你的 github 用户名和密码，ssh 方式（git@...）可以省略掉输入用户名密码的步骤。

"branch"写 master，因为 git pages 要求在 master 分支上部署网页。

"user"和"email"是你的 git 信息，在用`hexo deploy`或者`hexo d`部署博客的时候，hexo 自动向 master 分支上提交的 git 信息就是用的这个。

## 分类和标签
Hexo NexT主题下默认有首页和归档两个菜单，我们还可以开启其他菜单项，比如分类、标签、关于等等。
在根目录下分别输入命令：
```bash
hexo new page "tags"
hexo new page "categories"
```
可以看到有如下运行结果：
```bash
$ hexo new page "tags"
INFO  Created: ~/hexo/source/tags/index.md
 
$ hexo new page "categories"
INFO  Created: ~/hexo/source/categories/index.md
```
根据上面的路径，找到.../categories/index.md这个文件，打开后默认内容是这样的：
```
---
title: categories
date: 2020-04-26 12:23:34
---
```
改成：
```
---
title: 文章分类
date: 2020-04-26 12:23:34
type: "categories"
---
```

找到.../tags/index.md这个文件，打开后默认内容是这样的：
```
---
title: tags
date: 2020-04-26 12:25:51
---
```
改成：
```
---
title: 标签
date: 2020-04-26 12:25:51
type: "tags"
---
```

可以发现，分类和标签的配置方法基本一样，这样就创建了分类和标签页，此时如果运行`hexo s`预览发现侧边栏没有分类和标签的选项，还要进行以下操作：
在 `/themes/next/_config.yml` 配置文件中搜索 "menu" 找到如下配置项，将 tags 和 categories 前的#号去掉，就开启了标签和分类页。
{% asset_img config.jpg '/themes/next/_config.yml' %}

## 使用 git 托管
完成了 hexo 的配置之后，还要将其上传到 github 托管，进入博客项目根目录，运行如下命令：
```bash
git init
git add -A
git commit -m "init"

<!--新建一个分支用于储存博客源代码，以后新建博客等操作都在这个分支下进行-->
git branch hexo-resource
git checkout hexo-resource

<!--删除 master 分支，以后在 hexo-resource 分支下运行 "hexo d" 命令时 hexo 会自动创建一个干净的 master 分支-->
git branch -d master

<!--将本地仓库与 github 远程仓库关联-->
git remote add origin git@github.com:username/username.github.io.git
git push -u origin hexo-resource
```

至此，hexo 博客的配置全部完成，可以开始写博客了。

## 写博客的工作流
### 创建新博客
在博客项目根目录下运行：
```bash
hexo new title
```
创建文件名为 title 的新博客文件，这个文件路径为 `source/_posts/title.md`，打开便可编辑博客。终端输入`hexo server` 或 `hexo s` 即可开启 hexo 本地预览服务。
### 添加分类和标签属性
在文章头部添加tags和 categories 字段，形如：
```
---
title: Mac 上配置 Hexo
date: 2020-04-26 12:12:57
categories: 
- Web前端
tags:
- Mac
- hexo
---
```
注意，tags 可以有多个，categories 只能有一个。

### 文档标记
`<!--more-->` 标记之后的会在列表页隐藏

```
---
title: Mac 上配置 Hexo
date: 2020-04-26 12:12:57
---
```
其中 date 字段决定文档创建时间，hexo 会按照这个时间将文档归档。

### 部署
```bash
hexo g//在 public 文件夹生成博客网页文件
hexo d//部署博客
```
也可以用以下两种方式，他们的作用一样：
```bash
hexo g -d
hexo d -g
```

## 坑
### 更换主题后本地预览正常，部署到线上主题还是以前的
等一会就好了。。。

### hexo d 部署后提交记录是全局的 git 信息而非仓库的
打开根目录下 `_congif.yml` 文件，搜索"deploy"，改成类似下面这样的：
```yml
deploy:
  type: git
  repo: git@github.com:username/username.github.io.git
  branch: master
  user: nickname
  email: email@gmail.com
```
"user"和"email"是你的 git 信息，在用`hexo deploy`或者`hexo d`部署博客的时候，hexo 自动向 master 分支上提交的 git 信息就是用的这个。
当然你也可以不写这两行，hexo 会使用全局的 git --config 信息，但是因为我有两套 git 配置，一套全局的用于提交工作代码，一套私人的用于往 github 上提交自己的代码，私人代码我每次都会重设当前仓库的git 信息，但是貌似 hexo 读取的是全局的而不是仓库的 git 信息，这导致我 hexo-resource 分支上的提交记录和 master 分支的提交记录是两个人的（笑。故而我在配置文件里显式标明 git 信息。
