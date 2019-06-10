---
title: Linux防火墙常用命令
date: 2019-06-10 17:20:31
tags: [linux, 防火墙]
categories : Linux
---
linux系统常用的防火墙分别是iptables和firewall防火墙，记录一下常用的命令
<!--more-->
### iptables防火墙
```bash
#查看防火墙状态
service iptables status  

#停止防火墙
service iptables stop  

#启动防火墙
service iptables start  

# 重启防火墙
service iptables restart  

# 永久关闭防火墙
chkconfig iptables off  

# 永久关闭后重启
chkconfig iptables on
```

### firewall防火墙
```bash
#查看firewall服务状态
systemctl status firewalld

#查看firewall的状态
firewall-cmd --state

# 开启
service firewalld start
# 重启
service firewalld restart
# 关闭
service firewalld stop

#查看防火墙规则
firewall-cmd --list-all 

# 查询端口是否开放
firewall-cmd --query-port=8080/tcp
# 开放80端口
firewall-cmd --permanent --add-port=80/tcp
# 移除端口
firewall-cmd --permanent --remove-port=8080/tcp

#重启防火墙(修改配置后要重启防火墙)
firewall-cmd --reload
```

参数解释
firwall-cmd：是Linux提供的操作firewall的一个工具；
--permanent：表示设置为持久；
--add-port：标识添加的端口；