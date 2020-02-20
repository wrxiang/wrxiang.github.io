---
title: Zookeeper实现服务注册与发现
date: 2020-02-20 14:21:29
categories: Java
tags: [zookeeper]
---
&emsp;&emsp;在分布式架构中，系统经常被暴漏为服务供其他系统调用，为使系统间能够相互通信，需要有一个协调系统来管理这些服务，这就是服务注册与服务发现，这个协调者也被称之为注册中心。
下面基于zookeeper来实现一个简单的服务注册与发现功能
<!--more-->
通过Maven引入需要的jar包
```
<dependency>
  <groupId>com.101tec</groupId>
  <artifactId>zkclient</artifactId>
  <version>0.11</version>
</dependency>
```
创建服务注册类
```java
package com.wrxiang.zk.util;

import org.I0Itec.zkclient.ZkClient;

public class ZkServiceRegistry {

    private ZkClient zkClient;

    public void registry(String serviceName, String serviceAddress){

        zkClient = new ZkClient("127.0.0.1");

        //创建registry节点
        String registryAddress = "/services";
        if(!zkClient.exists(registryAddress)){
            zkClient.createPersistent(registryAddress);
            System.out.println("创建服务注册节点");
        }

        //创建service节点
        String servicePath = registryAddress + "/" + serviceName;
        if(!zkClient.exists(servicePath)){
            zkClient.createPersistent(servicePath);
            System.out.println("创建服务节点，服务名：" + serviceName);
        }

        //创建address节点（临时）
        String addressPath = servicePath + "/address-";
        String addressNode = zkClient.createEphemeralSequential(addressPath, serviceAddress);
        System.out.println("创建服务地址节点：" + addressNode);
    }
}
```
创建服务发现类
```java
package com.wrxiang.zk.util;

import org.I0Itec.zkclient.ZkClient;
import org.springframework.util.CollectionUtils;

import java.util.Collection;
import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;
import java.util.concurrent.ThreadLocalRandom;

public class ZkServiceDiscovery {

    private final List<String> addressCache = new CopyOnWriteArrayList<>();

    private ZkClient zkClient;

    public String discovery(String serviceName){

        zkClient= new ZkClient("127.0.0.1");

        String servicePath =  "/services/" + serviceName;
        if(!zkClient.exists(servicePath)){
            throw new RuntimeException("未发现服务节点：" + servicePath);
        }

        String address;

        //先从本地系统缓存中取
        if(addressCache.size() == 1){
            address = addressCache.get(0);
        }else if(addressCache.size() > 1){
            address = addressCache.get(ThreadLocalRandom.current().nextInt(addressCache.size()));
        }else{
            //从ZK注册中心中取
            List<String> addressList = zkClient.getChildren(servicePath);
            addressCache.clear();
            addressCache.addAll(addressList);

            zkClient.subscribeChildChanges(servicePath,(parentPath,currentChilds)->{
                System.out.println("监听服务节点发生变化：" + parentPath);
                addressCache.clear();
                addressCache.addAll(currentChilds);
            });

            if(CollectionUtils.isEmpty(addressCache)){
                throw new RuntimeException("未发现有效的服务节点");
            }

            if(addressList.size() == 1){
                address = addressList.get(0);
            }else {
                address = addressList.get(ThreadLocalRandom.current().nextInt(addressList.size()));
            }
        }

        //获取实际地址
        String addressPath = servicePath + "/" + address;
        String hostAndPort = zkClient.readData(addressPath);

        System.out.println("获取到实际访问地址：" + hostAndPort);
        zkClient.close();
        return hostAndPort;
    }
}
```
创建测试类
```java
package com.wrxiang.zk.util;

public class ZookeeperApplicationTest {

    public static void main(String[] args) throws InterruptedException {

        String serviceName = "test.com";

        ZkServiceRegistry registry = new ZkServiceRegistry();
        registry.registry(serviceName,"192.168.3.1:8080");

        Thread.sleep(1000);

        ZkServiceDiscovery zkServiceDiscovery = new ZkServiceDiscovery();
        zkServiceDiscovery.discovery(serviceName);

    }
}
```
启动zookeeper，然后运行测试类，可以发现服务能够正常注册到zookeeper，也能够被正常的发现。



