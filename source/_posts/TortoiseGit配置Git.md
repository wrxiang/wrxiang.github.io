---
title: TortoiseGit配置Git
date: 2021-08-15 12:35:34
categories: git
tags: [TortoiseGit, git]
---
>Git-2.32.0-64
>TortoiseGit 2.12.0.0

<!--more-->
## 安装相应的软件
1. 下载和安装Git，下载地址[git](https://git-scm.com/download/win)
2. 下载和安装TortoiseGit，下载地址[TortoiseGit](https://tortoisegit.org/download/)

## 配置
### 常规配置
桌面上邮件菜单，选择 **TortoiseGit** -> **设置**，配置Git的用户名以及邮箱，如下图所示：
![20210815124800.png](https://i.loli.net/2021/08/15/sUDG6ZnMkSPHvNi.png)
![20210815124959.png](https://i.loli.net/2021/08/15/fepXZW3cMA8BSPY.png)

### 生成公钥和密钥
进入TortoiseGit安装目录下的bin目录，双击 `PuTTYgen.exe` 进入PuTTY Key Generator，操作步骤如下
1. 点击**Generate**按钮生成公钥，注意生成过程中鼠标不要划过进度条，有可能会导致进度条停止不动
2. 点击**Save public key**，保存生成的公钥
3. 点击**Save private key**，保存生成的私钥

![20210815125911.png](https://i.loli.net/2021/08/15/9OyrYEb6gDABLRM.png)

### 配置密钥
进入TortoiseGit安装目录下的bin目录，双击 `pageant.exe`，如果提示已运行，在托盘程序中找到该程序双击一下即可
点击 **Add Key**按钮添加上一步中生成的私钥文件即可

![20210815130506.png](https://i.loli.net/2021/08/15/sjWo1xa3FDtBAZL.png)
### 配置公钥
在代码托管系统中配置公钥，下面以github为例
在GitHub设置中选择 **SSH and GPG kes**，添加SSH key
![20210815131153.png](https://i.loli.net/2021/08/15/OWJMZKbenqjN3YT.png)
选择**New SSH key**，将title以及公钥信息填写进去，点击保存即可
![20210815131530.png](https://i.loli.net/2021/08/15/cveXY2I5uhmFDTs.png)

## 代码同步
在文件夹中右键选择 **Git 克隆**，填写代码库的地址以及本地文件夹地址，并且选择 **加载Putty密钥**选项，并且选择生成的私钥文件(后缀名为.ppk的文件)，点击确定后将代码拉取到本地，之后更改后进行上传推送即可
![20210815131936.png](https://i.loli.net/2021/08/15/MmxIlt16wo3PWqV.png)




