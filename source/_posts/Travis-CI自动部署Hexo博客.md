---
title: Travis CI自动部署Hexo博客
date: 2019-06-03 21:02:36
categories: Hexo
tags: [Travis CI, hexo]
---
为了方便博客管理，我是将博客源码放在`resource`分支，最终生成部署的页面文件放在`master`分支，每次更新博客都需要先提交博客文件，然后手动部署到GiHub Pages，次数一多就显得麻烦，而且还容易出错。那么有没有一种方法让这个过程自动化，在提交完博客文件后自动帮我们编译部署文件到GitHub上呢？很幸运有现成的工具可供使用，使用Travis-CI就可以将整个过程自动化，下面借用一张图片来说明Travis_CI的作用

<img src="https://i.loli.net/2019/06/03/5cf52f17dff4a40269.png"/>

<!--more-->

## 准备工作
* 一个Hexo源码仓库，源码放在`resource`分支，编译后的文件放在master分支，类似于下面这样：
<img src="https://i.loli.net/2019/06/03/5cf5210ccb58054595.png" />

* 获取GitHub_Token
登录GitHub，在`Settings / Developer settings / Personal access tokens`中，新建一个token，权限将repo选中就可以，生成的token一定要保存好，等会配置在Travis中，只会显示一次，丢失就需要再重新生成一个。
<img src="https://i.loli.net/2019/06/03/5cf52393ac40687309.png" />
<img src="https://i.loli.net/2019/06/03/5cf5245446d5725695.png" />

* 进入[Travis CI官网](https://travis-ci.org/)，设置权限
Travis可以使用GitHub账号进行登录，进入设置（Settings），打开需要自动编译的公开仓库
<img src="https://i.loli.net/2019/06/03/5cf52592ad01949949.png" />
然后点击右侧的Settings按钮，找到环境变量(Environment Variables)，左侧填一个名字，右侧填写刚刚获取到的Github_Token，`Display value in build log`按钮不要打开，打开后其他人就可以在日志文件中看到你的Github_Token
<img src="https://i.loli.net/2019/06/03/5cf526cad28ae35157.png" />

## 告诉Travis怎样编译我们的项目
在Hexo站点目录，也就是`.config.yml`文件同级目录，创建一个名为`.travis.yml`的文件

```
language: node_js # 语言
node_js: stable # node.js版本
cache:          # 缓存，加快下次编译速度
  directories:
    - node_modules

branches:      # 监控 resource分支，此分支发生变化就启动一次新的构建
  only:
    - resource

before_install:  # 和自己电脑上安装的hexo-cli一样，也需要安装依赖环境
  - npm install -g hexo-cli

install:
  - npm install
  - npm install hexo-deployer-git --save
    
script:        # 编译Hexo
  - hexo clean
  - hexo generate

after_script:  # 编译完成后推送到GitHub上
  - git config user.name "wrxiang"  
  - git config user.email "wrxiang@163.com"
  
  #将.config.yml中的文字`Travis`替换为环境变量中的{Travis}代表的那串密码
  - sed -i "s/Travis/${Travis}/g" ./_config.yml  
  - hexo deploy
```
我们还需要修改_config.yml文件的deploy节点
```
#修改前
deploy:
  - type: git
    repo: git@github.com:wangruix/wangruix.github.io.git
    branch: master
```
```
#修改后
deploy:
- type: git
  # 下方的gh_token会被.travis.yml中sed命令替换
  repo: https://Travis@github.com/wangruix/wangruix.github.io.git
  branch: master
```

最后将新创建的`.travis.yml`文件提交到源码路径
<img src="https://i.loli.net/2019/06/03/5cf529f8763d858926.png" />

我们就可以新增一篇博客，完成后将博客提交到源码分支，然后就可以到[Travis](https://travis-ci.org/)查看构建情况，Job log中可以查看详细的构建日志
<img src="https://i.loli.net/2019/06/03/5cf52b21294dd75749.png" />

Travis构建完成后会将生成的文件推送到Github上，这样我们只需要提交博客的源文件，然后Travis帮我们完成构建部署过程，博客发布更新更加方便了。

## 参考文章 
[Travis CI使用经验](https://segmentfault.com/a/1190000016603414?utm_source=tag-newest)
[用TravisCI持续集成自动部署Hexo博客的个人实践](https://blog.csdn.net/qq_23079443/article/details/79015225)



