---
title: 指定tomcat运行时的JDK版本
date: 2019-06-04 13:55:28
tags: [tomcat, JDK]
categories : Java
---
tomcat作为开发中常用的web应用服务器，给开发和测试带来了很大的方便，tomcat运行依赖JDK的支持，在安装JDK时，经常会配置环境变量`JAVA_HOME`,用以指定所使用的JDK。然而有时候运行tomcat时需要依赖指定版本的JDK，我们又不能更改`JAVA_HOME`以免影响其他项目，下面将说明如何修改tomcat指定运行时的JDK版本。
<!--more-->
## Windows
编辑`tomcat\bin\setclasspath.bat`文件，在文件的开始添加如下设置
```bash
set JAVA_HOME=D:\devlop\Java\jdk7\jdk1.7.0_51
set JRE_HOME=D:\devlop\Java\jdk7\jre7
```
## Linux
编辑`tomcat\bin\setclasspath.sh`，在文件的开始添加如下设置
```bash
export JAVA_HOME=/usr/local/Java/jdk7/jdk1.7.0_51
export JRE_HOME=/usr/local/Java/jdk7/jre7
```
上面的意思是设定JAVA_HOME和JRE_HOME的路径，修改setclasspath文件后，tomcat在启动时便使用我们设定的JDK。

## 追根溯源
tomcat在启动的时候是运行bin目录下的startup.bat，他会调用catalina.bat，而catalina.bat会调用setclasspath.bat文件来获取JAVA_HOME和JRE_HOME这两个环境变量的值，因此若要在tomcat启动时指向特定的JDK，则需在setclasspath.bat文件的开头处加上JAVA_HOME和JRE_HOME。

从启动流程上来看，还有其他修改方式，如下：
修改`tomcat/bin/catalina`，同样根据操作系统增加上面的配置。

这两种方式任意一种都可以实现不配置环境变量，修改tomcat依赖的JDK环境。
