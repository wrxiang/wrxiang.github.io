---
title: Windows安装OpenSSH服务
date: 2020-01-18 20:38:30
categories: 持续集成
tags: [OpenSSH, Windows]
---
最近在做Jenkins的持续集成，公司的应用服务器大多是Windows系统，平时的运维主要是靠远程桌面的方式，为了使用Jenkins自动部署，需要在windows系统上安装SSH的服务，这里介绍微软官方的解决方案，详细介绍可以参考官网wiki。
基于PowerShell的OpenSSH: [https://github.com/PowerShell/Win32-OpenSSH/releases](https://github.com/PowerShell/Win32-OpenSSH/releases) 
<!--more-->
## 安装步骤
1. 进入链接下载最新的OpenSSH-Win64.zip，解压至 `C:\Program Files\OpenSSH`
2. 打开cmd，进入 `C:\Program Files\OpenSSH`安装目录，执行以下命令
```bash
powershell.exe -ExecutionPolicy Bypass -File install-sshd.ps1
```
3. 设置服务自动启动并启动服务
```bash
sc config sshd start= auto
net start sshd
```
4. 修复主机端的文件权限,进入 `C:\Program Files\OpenSSH`，右键 FixHostFilePermissions.ps1【使用PowerShell运行】，命令行提示全选是

执行这一步时有可能命令窗口一闪而过，由于系统禁止执行脚本的原因，打开Shell窗口，执行命令 `set-executionpolicy remotesigned`修改策略即可

至此ssh服务安装完成，默认端口是22，若服务器上开启防火墙需要设置对该端口的允许



## 修改设置
进入 `C:\ProgramData\ssh` 目录下的sshd_config文件是ssh的配置文件，常用配置如下
```bash
# 端口号
Port 22
# 设置ssh在接收登录请求之前是否检查用户家目录和rhosts文件的权限和所有权
StrictModes no
# 密钥访问
PubkeyAuthentication yes
# 密码访问
PasswordAuthentication yes
# 空密码访问
PermitEmptyPasswords no
# administrator用户组的用户默认公钥文件路径
AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
```
进入`C:\ProgramData\ssh\`目录，创建administrators_authorized_keys文件，在该目录下打开cmd窗口，执行以下命令设置权限
```
icacls administrators_authorized_keys /inheritance:r
icacls administrators_authorized_keys /grant SYSTEM:(F)
icacls administrators_authorized_keys /grant BUILTIN\Administrators:(F)
```

cmd中重启sshd服务就可以使用密钥登录SSH服务
```
net stop sshd
net start sshd
```
## 踩入的坑
1. 网上很多教程介绍需要在 `C:\Users\账户名\.ssh`目录创建 `authorized_keys`公钥文件，后来发现7.9版本之后，公钥文件默认路径是 `C:\ProgramData\ssh\administrators_authorized_keys`并且需要设置权限，设置权限命令上文已有说明
2. Windows Service2012R2即使配置了公钥，连接时依然显示没有注册公钥，官方判断可能是权限问题 [Fix SSH file permissions](https://github.com/PowerShell/Win32-OpenSSH/wiki/OpenSSH-utility-scripts-to-fix-file-permissions)，解决方法：进入安装目录，右键 FixHostFilePermissions.ps1【使用PowerShell运行】，命令行提示全选是，重启sshd服务后密钥连接正常
3. 配置密钥登录后仍需要输入密码，SSH默认不希望home目录和~/.ssh目录对组有写权限，修改配置文件中`StrictModes no`，不让 sshd 去检查用户家目录或相关档案的权限数据，这里我们粗暴的设置为不检查权限就好，或者设置客户端的权限，若客户端是linux可参考以下命令
```
chmod g-w /home/wjw 
chmod 700 /home/wjw/.ssh
chmod 600 /home/wjw/.ssh/authorized_keys
```
