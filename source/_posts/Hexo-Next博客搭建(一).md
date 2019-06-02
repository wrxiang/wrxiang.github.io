---
title: Hexo-Next博客搭建(一)
date: 2019-05-01 13:21:25
categories: Hexo
tags: [hexo, next]
---

# 简介
[Hexo](https://hexo.io/zh-cn/) 是一个快速、简介并且高效的静态站点生成框架，他基于[Node.js](https://nodejs.org/en/)，可以快速的生成静态网页，支持Markdown解析文章，并且可以一键部署，只需要一条指令即可部署到GitHub Pages或其他网站。

通过Hexo这个博客框架，可以轻松的使用Markdown编写文章，除Markdown本身的语法外，还可以使用Hexo提供的[标签插件](https://hexo.io/zh-cn/docs/tag-plugins.html) 来快速的插入特定形式的内容。

基于Hexo这个框架，有很多优秀的主题可供使用，其中[Next](http://theme-next.iissnan.com/)因"精于心,简于形"的风格，一直被广大用户所喜爱。

<!-- more -->

# 运行环境安装
Hexo是基于Node.js的博客框架，使用之前需要先安装Node.js的运行环境，在[Node.js官网](https://nodejs.org/en/download)下载电脑系统对应的安装包，下载完成后运行开始安装Node.js，选择好安装路径，一路【Next】即可完成安装。

安装完成后在`cmd`命令窗口中进行简单的测试安装是否成功，后面还要进行环境变量的配置。
```bash
node -v  --输入node -v显示版本说明Node.js已安装成功
npm -v   --输入npm -v显示版本说明自带的npm已安装成功
```

这里的环境配置主要配置的是npm安装的全局模块所在的路径，以及缓存cache的路径，之所以要配置是因为在执行npm全局安装的时候，会将安装的模块安装到【C:\Users\用户名\AppData\Roaming\npm】下，占用C盘空间。一般是将全局模块和缓存放在Node.js的安装路径中。

例如我的Node.js安装根目录为【D:\Develop\nodejs】，在Node.js下创建两个文件夹：【node_global】和【node_cache】，创建完两个文件夹后，在`cmd`窗口输入以下命令设置全局模块及缓存的路径
```bash
npm config set prefix "D:\Develop\nodejs\node_global"
npm config set cache "D:\Develop\nodejs\node_cache"
```

接下来是配置环境变量，“我的电脑”-右键-“属性”-“高级系统设置”-“高级”-“环境变量”，打开环境变量对话框中，在【系统变量】下新建【Node_PATH】，变量值为【D:\Develop\nodejs\node_global\node_modules】，将【用户变量】下的【Path】修改为【D:\Develop\nodejs\node_global】

# 安装及配置Git
Hexo中可以使用Git进行博客的部署，所以需要安装Git，访问[git官网](https://git-scm.com/download/)下载电脑系统对应的git安装包，运行安装包，一路【Next】完成安装，安装好之后，配置用户名及邮箱，在安装目录打开`git-bash`：
```bash
# 配置全局用户名
git config --global user.name "yourname"
# 配置全局邮箱
git config --global user.email "youremail@qq.com"
```
用户名及邮箱和GitHub相同（如果没有去[GitHub](https://github.com)官网注册一个，注册完成需要邮箱验证才能正常使用）

配置`ssh`，安装目录打开`git-bash`：
```bash
#输入，回车
ssh-keygen
```
复制（右键+复制，不能ctrl+c，这里ctrl+c是结束命令的意思）命令中出现的的`/c/Users/Administrator/.ssh/id_rsa`，然后把它粘贴（右键+粘贴）到冒号后面，然后回车，回车，回车。。。直到结束（中间的冒号，除了第一个不用管只管回车）

本地生成ssh键值后，需要在GitHub上进行绑定，网页上打开github -> 点击头像 -> Settings -> 左边菜单找到 `SSH and GPG keys` -> `New SSH key`（绿色按钮）-> title随便填（用英文）-> key值（打开`/c/Users/Administrator/.ssh/id_rsa`所在的文件夹，找到`id_rsa.pub`，注意是pub后缀那个文件，用记事本打开，复制里面的内容，粘贴到key值。） -> 点击下面的`Add SSH key` -> 完成。

# 安装及使用Hexo
创建一个空白的文件夹（随意命名，这里以MyBlog为例），打开`cmd`命令窗口，切换到创建的空白文件夹中
```bash
# 切换国内源
npm config set registry="http://registry.cnpmjs.org"

# 安装git部署插件
npm install hexo-deployer-git save

# 安装hexo
npm install -g hexo

# 初始化Hexo
hexo init

# 安装必要模块
npm install
```
命令执行完成之后在创建的空白文件中会生成一些文件，

测试一下：
```bash
# 生成静态文件
hexo g

# 运行服务，本地测试
hexo s
```

浏览器上输入[http://localhost:4000](http://localhost:4000)，可以看到Hexo默认主题的网页，这样Hexo博客就安装完成，后续可以对该博客更换主题，美化页面。

# 项目部署
通过以上操作，一个简单的Hexo博客已经搭建完成，那么如何将搭建的博客部署到GitHub上呢？
首先配置GitHub仓库，打开站点文件夹，编辑 `_config.yml`文件，
```
deploy:
  type: git
  repository: https://github.com/wrxiang/wrxiang.github.io.git #更改为你的博客仓库地址
  branch: master
```

配置完成后通过命令`hexo d`，将项目部署到GitHub仓库中，部署完成后就能够通过网址打开博客



# 总结
```bash
hexo clean #清除生成的静态页面
hexo g #生成静态页面
hexo s #启动hexo服务，开启预览访问端口，默认4000
hexo d #部署项目
```

Hexo中的图标使用的是[Font Awesome](https://fontawesome.com/?from=io)，所以博客中已经自带Font Awesome中所有的图标，基本可以满足我们的所有需要，可以去Font Awesome中查找想要使用图标。
<i class="fa fa-github"></i> `<i class="fa fa-github"></i>`
<i class="fa fa-github fa-lg"></i> `<i class="fa fa-github fa-lg"></i>`
<i class="fa fa-github fa-2x"></i> `<i class="fa fa-github fa-2x"></i>`