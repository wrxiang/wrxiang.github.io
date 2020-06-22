---
title: windosw系统配置nginx实现https访问
date: 2020-04-04 20:17:21
categories: nginx
tags: [nginx, 反向代理]
---

# 安装nginx
在[官网](http://nginx.org/en/download.html)下载和系统匹配的nginx，解压即可

# 安装OpenSSL
由于代理https需要使用证书，这里我们使用OpenSSL生成自签名的证书，可以下载与系统相匹配的OpenSSL程序进行安装，安装过程一直下一步即可，[下载地址](http://slproweb.com/products/Win32OpenSSL.html)
<!--more-->
# 配置环境变量
1. 新增环境变量 `OPENSSL_HOME`，变量值为 `D:\OpenSSL-Win64`，指向OpenSSL的安装路径
2. 新增环境变量 `OPENSSL_CONF`，变量值为 `D:\OpenSSL-Win64\bin\openssl.cfg`，指向openssl.cfg文件目录
3. 在环境变量path的末尾添加 `%OPENSSL_HOME%\bin`

# 生成证书
在nginx安装目录创建 `ssl`文件夹，用于存放证书，如我的是 `F:\nginx-1.16.1\ssl`，以管理员权限运行cmd并进入创建的ssl目录
## 创建私钥
在命令行中执行命令 `openssl genrsa -des3 -out demo.key 1024` ，demo是私钥文件名，可以自定义，此处需要输入和验证密码，输入的密码请记住，后面会用到，这里输入的密码是 `123456`
![1586003554_1_.png](https://i.loli.net/2020/04/04/fyEH1oQ7O5iPgub.png)

## 创建crs证书
在命令行中执行 `openssl req -new -key demo.key -out demo.csr`，其中key文件就是第一步生成的私钥，demo为自定义的文件名，如下所示，这里主要设置的是国家地区以及组织的一些信息，其中最重要的是 `Common Name`，这里输入的就是我们要用https访问的域名，也是nginx中所要配置的 `server_name`，完成所有步骤后，ssl文件夹中会生成demo.csr和demo.key两个文件
![1586004067_1_.png](https://i.loli.net/2020/04/04/uhPa4kM5n8V2ywS.png)

## 去除密码
在加载SSL支持的Nginx并使用上述私钥时除去必须的口令，否则会在启动nginx的时候需要输入密码。复制demo.key并重命名为demo.key.org。
在命令行中执行此命令：`openssl rsa -in demo.key.org -out demo.key`  如下图所示，此命令需要输入刚才设置的密码。

## 生成crt证书
在命令行中执行此命令：`openssl x509 -req -days 365 -in demo.csr -signkey demo.key -out demo.crt`

证书生成完毕，我们在ssl文件夹中可以看到生辰的证书文件，我们要用到的就是demo.crt和demo.key。

# 修改nginx配置
在nginx.conf配置文件中添加一个如下server
```
server {
        listen  443 ssl;
        server_name  www.demo.com;
        ssl_certificate      F:/nginx-1.16.1/ssl/demo.crt;
        ssl_certificate_key  F:/nginx-1.16.1/ssl/demo.key;

        location / {
          root   html;
          index  index.html index.htm;
        }
    }
```
# 启动nginx
正常启动nginx后，使用 https://127.0.0.1:443 访问，可以正常跳转
![20181031234338193.png](https://i.loli.net/2020/04/04/sVy5LqpgtQ6UGMC.png)



