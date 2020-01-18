---
title: linux主机通过ssh登录windows服务器启动tomcat所踩的坑
date: 2020-01-18 23:54:30
categories: 持续集成
tags: [jenkins, ssh]
---
## 情景描述
公司的Jenkins服务安装在linux系统上，但是很多应用服务器都是windows系统，那么在jenkisn上编译生成的war包如何传输到windows服务器上并且进行tomcat服务的重启呢？相对于linux的应用服务器，使用jenkins在windows上做自动发布还是遇到了一些不好解决的问题，windows系统在运维上还是有许多不便的（默默吐槽一句）。
<!--more-->
## 发布步骤
类比linux的应用服务器，首选是通过ssh远程登录的方式进行，这样的话jenkins的配置都不需要怎么更改，简单的发布步骤如下：
1. jenkins中使用MAVEN编译打包，生成所需的war包
2. scp将war包传输到应用服务器
3. ssh远程执行应用服务器的部署重启脚本
4. 发布完成

windows默认是不支持ssh服务的，这里我采用的是微软官方的解决方案OpenSSH，安装步骤可以参考：[Windows安装OpenSSH服务](https://wrxiang.github.io/2020/01/18/Windows%E5%AE%89%E8%A3%85OpenSSH%E6%9C%8D%E5%8A%A1)

## 遇到的问题
jenkisn远程调用应用服务器上的部署脚本后，tomcat服务没有启动，但是在应用服务器上手动运行脚本是能够完成项目的部署以及重启的，通过调试发现远程调用bat脚本确实将tomcat服务停掉并进行了重启，但是随着ssh连接的断开，通过ssh启动的进程也一同销毁。
通过搜索问题发现，在ssh到windows的情况下，只要ssh退出，ssh启动的任务都会被kill掉。万事具备，只欠重启，这步实现不了，那之前的工作和没有做一样，没办法，只能找各种方案进行尝试。

## 尝试过的方案
### 防止jenkins杀掉进程
执行脚本前增加 `export BUILD_ID=notkillme`，告诉jenkins任务结束后不要杀掉通过它启动的进程。jenkins和tomcat在同一台服务器上可以实现，但这是ssh断开的服务和jenkins无关。
### shell脚本
在windows 上安装linux的shell环境，我是通过安装一个git，然后将git的cmd和bin配置到环境变量path中来实现的。
shell中可以使用nohup命令启动tomcat，nohup 命令会忽略所有挂断（SIGHUP）信号。在注销后使用 nohup 命令运行的程序不会注销，仍会运行。
但是ssh远程启动脚本的情况下，ssh连接断开，通过nohup启动的服务也会注销
```shell
nohup ./startup.sh &  
```
### 将tomcat注册为服务
通过将tomcat注册成服务，在bat脚本中通过服务的方式启动，在ssh断开之后服务能够正常运行。
```
# 停止tomcat服务
net stop tomcat7
# 启动tomcat服务
net start tomcat7
```
通过这种方式，能够保证tomcat在ssh断开后仍能够运行，但是实践中发现我们项目不止要使用tomcat,还依赖一些中间件，例如zookeeper，MQ等，他们在windows下不能被注册为服务（zookeeper虽然可以注册为服务，但是都不建议在生产环境运行），此种方式使用场景有限制。

### 调用windows上的执行计划
在windows上创建一个执行计划，执行计划调用部署重启的脚本，ssh远程启动这个执行计划，通过这种方式触发的脚本能够保证在ssh断开后，服务一直可以运行，和手动执行脚本没有区别，此方式实际项目中使用。
```
# 启动名为DEPLOY的执行计划
schtasks /Run /TN "DEPLOY"
```



