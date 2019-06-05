---
layout: post
title: "Hexo Build Blog Notes"
subtitle: "Use Hexo to build Blog"
date: 2018-12-17
author: ppbibo
category: Notes
finished: true
---
[TOC]



# 博客搭建

在Centos 下搭建Hexo 静态博客托管到 Github上. [DEMO](https://ppbibo.github.io/)

现在以移植为 Jekyll 框架。



## 1.安装 Node.js

### 下载Node.js

```
$ wget https://nodejs.org/dist/v4.4.4/node-v4.4.4-linux-x64.tar.xz
```

### 解压到当前文件夹

```
$ tar -xvJf node-v4.4.4-linux-x64.tar.xz
$ mv node-v4.4.4-linux-x64 node-v4.4.4
$ mv node-v4.4.4 /usr/local/node
```

### 配置环境变量

```
# 编辑 /etc/profile
$ vim /etc/profile
# 在底部添加 PATH 变量
$ export PATH=$PATH:/usr/local/node/bin
# 保存退出
$ wq
# 最后保存并使其生效即可
```

## 2.安装 hexo

```
# 创建目录
$ mkdir hexo
# 切换目录
$ cd hexo
# 安装Git(已安装可跳过)
$ yum install git-core
# 安装 Hexo
$ npm install -g hexo-cli
# 初始化 Hexo
$ hexo init

# 生成后的文件目录
node_modules：是依赖包

public：存放的是生成的页面

scaffolds：命令生成文章等的模板

source：用命令创建的各种文章

themes：主题

_config.yml：整个博客的配置

db.json：source解析所得到的

package.json：项目所需模块项目的配置信息
```

### 安装插件

```
$ npm install hexo-generator-index --save
$ npm install hexo-generator-archive --save
$ npm install hexo-generator-category --save
$ npm install hexo-generator-tag --save
$ npm install hexo-server --save
$ npm install hexo-deployer-git --save
$ npm install hexo-deployer-heroku --save
$ npm install hexo-deployer-rsync --save
$ npm install hexo-deployer-openshift --save
$ npm install hexo-renderer-marked --save
$ npm install hexo-renderer-stylus --save
$ npm install hexo-generator-feed --save
$ npm install hexo-generator-sitemap --save
$ npm install hexo-asset-image --save
```

### hexo配置文件修改

```
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: bibo @ 安全小生 
subtitle: know it then do it.
description: 安全小生
author: ppbibo
language: zh-CN
timezone:

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://www.6o9.im
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:

# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:

# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: true
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:
  
# Home page setting
# path: Root path for your blogs index page. (default = '')
# per_page: Posts displayed per page. (0 = disable pagination)
# order_by: Posts order. (Order by date descending by default)
index_generator:
  path: ''
  per_page: 10
  order_by: -date
  
# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination
## Set per_page to 0 to disable pagination
per_page: 20
pagination_dir: page

# Extensions
#
#
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:ppbibo/ppbibo.github.io.git
  branch: master
  message: '站点更新:{{now("YYYY-MM-DD HH/mm/ss")}}'

# RSS订阅
feed:
 type: atom
 path: atom.xml
 limit: 20
 hub:
 content:
 content_limit: 140
 content_limit_delim: ' '
```

## 配置Github

### 1.配置全局git环境

```
$ git config --global user.email "you@example.com"
$ git config --global user.name "Your Name"
```

### 2.生成SSH密钥

```
$ ssh-keygen -t rsa -C you@example.com
```

### 3.在Github上面创建创建博客

创建一个名称为 “yourusername.github.io” 的新项目

### 4.将ssh秘钥添加到github中

将秘钥放到github上去,登录你的github账号,
进入[秘钥设置面板](https://github.com/settings/ssh),
执行

```
$ cat ~/.ssh/id_rsa.pub
```



将里面的密码复制到github上,点击 Add key

### 5.配置hexo

更改hexo配置文件，repo 为你你自己的博客。

```
deploy:
  type: git
  repo: git@github.com:ppbibo/ppbibo.github.io.git. 
  branch: master
  message: '站点更新:{{now("YYYY-MM-DD HH/mm/ss")}}'
```



执行即可部署你的博客.

```
$ hexo d -g
```



### 6.修改主题

在hexo 目录下

```
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```



修改在hexo目录下的_config.yml ，找到片 theme: ，默认为 landscape，改为

```
theme: theme: next
```



### 7.hexo常用命令

hexo server -s #静态模式
hexo server -p 5000 #更改端口
hexo server -i 192.168.1.1 #自定义 IP
hexo clean #清除缓存 网页正常情况下可以忽略此条命令
hexo g #生成静态网页
hexo d #开始部署
hexo new “postName” #新建文章
hexo new page “pageName” #新建页面
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口（默认端口4000，’ctrl + c’关闭server）
hexo deploy #将.deploy目录部署到GitHub

### 8.一些扩展

[next的使用说明](http://theme-next.iissnan.com/getting-started.html)
[修改Hexo主题](https://www.jianshu.com/p/33bc0a0a6e90)
[hexo 文章写法](https://blog.csdn.net/weixin_42796909/article/details/81745271)
[其他主题](https://hexo.io/themes/)