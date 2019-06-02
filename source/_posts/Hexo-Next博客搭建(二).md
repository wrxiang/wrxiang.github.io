---
title: Hexo-Next博客搭建(二)
date: 2019-05-01 16:21:26
categories: Hexo
tags: [hexo, next]
---
基于Hexo博客框架，有很多优秀的主题可以使用，其中[Next](http://theme-next.iissnan.com/)主题因其简洁美观的风格收到广大用户的喜爱，本篇文章将介绍基于Next主题的一些基础配置及拓展功能的实现。

本文中所使用的Next主题版本为[5.1.4](https://github.com/iissnan/hexo-theme-next/releases/tag/v5.1.4)，PS：此版本已不再维护，社区维护的版本在这里[Next v6和v7](https://github.com/theme-next/hexo-theme-next)

{% note info %}在Hexo中拥有两份重要的配置文件，其名称都是_config.yml。其中一份位于站点目录下，主要包含Hexo本身的配置，另一份位于主题目录下，这份配置由主题作者提供，主要用于配置主题相关的选项。
我们约定，将前者称为<span id="inline-blue">站点配置文件</span>，后者称为<span id="inline-purple">主题配置文件</span>{% endnote %}

<!--more-->

## 设置头像
编辑 <span id="inline-purple">主题配置文件</span>，修改avatar字段，值设置成头像的链接地址，头像图片可放置在站点目录`source/images`目录下
设置如下：`avatar: /images/avatar.gif`

## 设置站点个人信息
```
title: 标题
subtitle: 副标题
description: 描述
keywords:
author:  作者昵称
```

## 设置Menu
<span id="inline-purple">主题配置文件</span>中将所需要的菜单注释放开即可
```
menu:
  home: / || home
  about: /about/ || user
  tags: /tags/ || tags
  categories: /categories/ || th
  archives: /archives/ || archive
```
新添加的菜单点击会报错，例如：Cannot GET /about/
在站点目录下使用hexo命令创建新增菜单的文件夹
```bash
hexo new page "tags"
hexo new page "about"
hexo new page "archives"
hexo new page "categories"
```
命令执行后会在站点目录`source`目录下创建对应的文件夹及相应的文件

## 修改标签及分类页面
编辑`source\tags\index.md`
```
---
title: 标签
date: 2019-05-20 21:00:45
type: "tags"
---
```

编辑`source\categories\index.md`
```
---
title: 分类
date: 2019-05-20 21:01:36
type: "categories"
---
```



## 修改内容宽度
Next 对内容的宽度的设定如下：
- 700px，当屏幕宽度 < 1600px
- 900px，当屏幕宽度 >= 1600px
- 移动设备下，宽度自适应
如果你需要修改内容的宽度，同样需要编辑样式文件。编辑主题的 source\css_variables\custom.styl 文件，新增变量：

```css
// 修改成你期望的宽度
$content-desktop = 700px

// 当视窗超过 1600px 后的宽度
$content-desktop-large = 900px

```

但是该方法不适用于 Pisces，Gemini风格，编辑 `themes\next\source\css\_schemes\Picses\_layout.styl`文件，在文件末尾新增以下代码
```css
// 以下为新增代码
header{ width: 90% !important; }
header.post-header {
  width: auto !important;
}
.container .main-inner { width: 90%; }
.content-wrap { width: calc(100% - 260px); }

.header {
  +tablet() {
    width: auto !important;
  }
  +mobile() {
    width: auto !important;
  }
}

.container .main-inner {
  +tablet() {
    width: auto !important;
  }
  +mobile() {
    width: auto !important;
  }
}

.content-wrap {
  +tablet() {
    width: 100% !important;
  }
  +mobile() {
    width: 100% !important;
  }
}

```

## 添加RSS
站点目录下安装hexo插件
```bash
npm install --save hexo-generator-feed
```
安装成功后编辑<span id="inline-blue">站点配置文件</span>，在末尾添加

```
# Extensions
## Plugins: http://hexo.io/plugins/
plugins: hexo-generate-feed
feed:
type: atom
path: atom.xml
limit: 20
```
编辑 <span id="inline-purple">主题配置文件</span>，修改rss字段
```
# Set rss to false to disable feed link.
# Leave rss as empty to use site's feed link.
# Set rss to specific value if you have burned your feed already.
rss: /atom.xml 
```


## 添加顶部进度条
编辑 <span id="inline-purple">主题配置文件</span>，修改`pace: true`开启顶部进度条，然后pace_theme设置一个喜欢的进度条样式
```
# Progress bar in the top during page loading.
pace: true
# Themes list:

pace_theme: pace-theme-center-simple
```


## 添加侧边栏社交小图标
编辑 <span id="inline-purple">主题配置文件</span>，修改`social`字段，打开对应的注释，修改为自己的链接
```
social:
  GitHub: https://github.com/wrxiang || github
  E-Mail: mailto:rxhome@163.com || envelope
```
设定链接的图标，对应的字段是 social_icons。其键值格式是 匹配键: Font Awesome 图标名称， 匹配键 与上一步所配置的链接的 显示文本 相同（大小写严格匹配），图标名称 是 Font Awesome 图标的名字（不必带 fa- 前缀）。 enable 选项用于控制是否显示图标，你可以设置成 false 来去掉图标。
```
# Social Icons
social_icons:
  enable: true
  # Icon Mappings
  GitHub: github
  Twitter: twitter
  微博: weibo
```



## 设置代码高亮显示
NexT 使用 Tomorrow Theme 作为代码高亮，共有5款主题供你选择。 NexT 默认使用的是 白色的 normal 主题，可选的值有 normal，night， night blue， night bright， night eighties：
更改 highlight_theme 字段，将其值设定成你所喜爱的高亮主题，例如：
```
# Code Highlight theme
# Available value:
#    normal | night | night eighties | night blue | night bright
# https://github.com/chriskempson/tomorrow-theme
highlight_theme: night bright
```

## 开启打赏
<span id="inline-purple">主题配置文件</span> 中添加支付宝或微信的二维码地址即可，将二维码图片放入`hemes/next/source/images`目录下或者使用图床路径
```
reward_comment: 坚持原创技术分享，您的支持将鼓励我继续创作！
wechatpay: /images/wechatpay.jpg
alipay: /images/alipay.jpg
```

## 添加站内搜索
NexT主题支持集成 Swiftype、 微搜索、Local Search 和 Algolia。下面使用Local Search，安装hexo插件
```
npm install hexo-generator-search --save
npm install hexo-generator-searchdb --save

```

编辑<span id="inline-blue">站点配置文件</span>，添加以下内容
```
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```
编辑 <span id="inline-purple">主题配置文件</span>，设置`Local search`属性为`true`


## 添加访客人数和总访问量
Next已内置不蒜子计数，编辑`/theme/next/_config.yml`,配置如下
```
# Show PV/UV of the website/page with busuanzi.
# Get more information on http://ibruce.info/2015/04/04/busuanzi/
busuanzi_count:
  # count values only if the other configs are false
  enable: true
  # custom uv span for the whole site
  site_uv: true
  site_uv_header: 您是第
  site_uv_footer: 个小伙伴
  # custom pv span for the whole site
  site_pv: true
  site_pv_header: 本站总浏览
  site_pv_footer: 次
  # custom pv span for one page only
  page_pv: false
  page_pv_header: <i class="fa fa-file-o"></i>
  page_pv_footer:
```

将`enable`的值由`false`改为`true`，便可以看到页脚出现访问量，上述配置表示：

`site_uv`表示是否显示整个网站的UV数
`site_pv`表示是否显示整个网站的PV数
`page_pv`表示是否显示每个页面的PV数

上述配置完成之后访问人数和总访问量并没有出现，是由于不蒜子计数网址更新导致Hexo Next统计浏览数失效，解决方法
编辑`\themes\next\layout\_third-party\analytics\busuanzi-counter.swig`，
将`src="https://dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js"`
替换成`src="https://busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"`

本地运行时，由于统计的是localhost的访问地址，访问人数及总访问量会比较大，发布到网站会正常

## 隐藏底部的  由 Hexo 强力驱动  主题 - NexT.Pisces  
<span id="inline-purple">主题配置文件</span> 设置如下
```
footer:
	powered: false
	theme:
		enable: false
```

## 站点底部图标更改为心形
<span id="inline-purple">主题配置文件</span> 的`foot`节点下`icon`更改为`heart`


## 启用字数统计、阅读时长
安装hexo插件`npm i --save hexo-wordcount`
<span id="inline-purple">主题配置文件</span>，修改配置如下：
```
# Post wordcount display settings
# Dependencies: https://github.com/willin/hexo-wordcount
# 文章字数展示设置
post_wordcount:
  # 文本显示
  item_text: true
  # 文章字数统计
  wordcount: true
  # 阅读时长
  min2read: true
  # 站点总字数统计
  totalcount: false
  # 该post_wordcount的所有设置另起一行显示
  separated_meta: true
```

## 文章底部添加版权信息
编辑 <span id="inline-purple">主题配置文件</span>，修改配置如下
```
# Declare license on posts
post_copyright:
  enable: true
  license: CC BY-NC-SA 3.0
  license_url: https://creativecommons.org/licenses/by-nc-sa/3.0/
```

[Next 使用文档](http://theme-next.iissnan.com/)

