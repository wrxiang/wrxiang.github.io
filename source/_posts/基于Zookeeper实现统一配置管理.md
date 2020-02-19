---
title: 基于Zookeeper实现统一配置管理
date: 2020-02-19 22:56:12
categories: java
tags: [zookeeper, 配置管理]
---
## 为什么需要统一配置？
&emsp;&emsp;我们在开发系统时，一般会将一些信息添加到配置文件中，比如数据库信息、日志等，如果需要更改也是修改配置文件然后再发布到生产中，这种方式在集群的环境下就会显得很麻烦。那么有什么解决方法呢？
1. 将公共配置抽取出来
2. 提供统一的配置入口对公共配置进行修改
3. 修改后的内容能够同步到各集群系统中
<!--more-->

## Zookeeper方案
&emsp;&emsp;这里我们采取zookeeper实现统一配置管理，配置信息存放在zookeeper的节点中，一旦配置信息发生改变，zk会通知监听该节点的客户端系统，客户端系统获取最新的配置信息，然后进行配置信息的重新载入。
下面基于zookeeper实现了一个粗略的统一配置管理
通过Maven引入zkclient.jar
```
<dependency>
  <groupId>com.101tec</groupId>
  <artifactId>zkclient</artifactId>
  <version>0.11</version>
</dependency>
```
创建配置信息实体 Config.java
```java
package com.wrxiang.zk.util;

import java.io.Serializable;

public class Config implements Serializable {

    private static final long serialVersionUID = -3960940730727782588L;

    private String userName;
    private String password;

    public Config(String userName, String password){
        this.userName = userName;
        this.password = password;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public String toString() {
        return "Config{" +
                "userName='" + userName + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}
```
配置管理中心ZkConfigMag
```java
package com.wrxiang.zk.util;

import org.I0Itec.zkclient.ZkClient;

public class ZkConfigMag {

    private Config config;

    /**
     * 模拟获取配置文件，从数据库加载或从前台获取
     * @return
     */
    public Config downLoadConfig(){
        config = new Config("test", "123");
        return config;
    }

    /**
     * 更新配置信息
     * @param userName
     * @param password
     */
    public void upLoadConfig(String userName, String password){
        if(config == null){
            config = new Config(userName, password);
        }else{
            config.setUserName(userName);
            config.setPassword(password);
        }
    }



    public void syncConfigToZk(){

        ZkClient zkClient = new ZkClient("127.0.0.1:2181");
        if(!zkClient.exists("/zkConfig")){
            zkClient.createPersistent("/zkConfig", true);
        }
        zkClient.writeData("/zkConfig", config);
        zkClient.close();
    }

}
```
应用监听实现ZkGetConfigClient
```java
package com.wrxiang.zk.util;

import org.I0Itec.zkclient.IZkDataListener;
import org.I0Itec.zkclient.ZkClient;

public class ZkGetConfigClient {

    private Config config;

    public Config getConfig() throws Exception {
        ZkClient zkClient = new ZkClient("127.0.0.1:2181");
        if(!zkClient.exists("/zkConfig")){
            throw new Exception("配置信息节点不存在");
        }
        config = zkClient.readData("/zkConfig");
        System.out.println("加载到配置信息：" + config.toString());

        zkClient.subscribeDataChanges("/zkConfig", new IZkDataListener() {
            @Override
            public void handleDataChange(String s, Object o) throws Exception {
                config = (Config)o;
                System.out.println("监听到配置信息被修改" + config.toString());
            }

            @Override
            public void handleDataDeleted(String s) throws Exception {
                config = null;
                System.out.println("监听到配置信息被删除");
            }
        });
        return config;
    }
}
```
测试，启动配置管理中心
```java
package com.wrxiang.zk.util;

public class ZkConfigTest {
    public static void main(String[] args) throws Exception {
        ZkConfigMag zkConfigMag = new ZkConfigMag();
        Config config = zkConfigMag.downLoadConfig();
        System.out.println("初始化加载配置信息：" + config.toString());
        zkConfigMag.syncConfigToZk();
        System.out.println("同步配置信息至zookeeper");

        ZkGetConfigClient zkGetConfigClient = new ZkGetConfigClient();
        zkGetConfigClient.getConfig();

        //暂停一会
        Thread.sleep(1000);

        zkConfigMag.upLoadConfig("test222", "3336666");
        System.out.println("修改配置文件：" + config.toString());
        zkConfigMag.syncConfigToZk();
        System.out.println("同步配置文件至zookeeper");

        Thread.sleep(1000);
    }
}

```

启动测试后，控制台输出
```
初始化加载配置信息：Config{userName='test', password='123'}
同步配置信息至zookeeper
加载到配置信息：Config{userName='test', password='123'}
修改配置文件：Config{userName='test222', password='3336666'}
同步配置文件至zookeeper
监听到配置信息被修改Config{userName='test222', password='3336666'}
```
通过上述例子介绍如何使用zokeeper实现简单的配置管理，实际系统应用肯定更加复杂，但是原理类似。
