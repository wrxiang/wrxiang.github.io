---
title: Spring中事件处理
date: 2019-10-30 09:51:23
categories: Spring
tags: [Spring, 事件处理]
---
Spring的核心是**ApplicationContext**，它负责管理beans的完整生命周期，当加载beans时，ApplicationContext发布某些类型的事件。例如，当上下文启动时，ContexttStartedEvent发布，当上下文停止时，ContextStoppedEvent发布。
通过ApplicationEvent类和ApplicationListener接口来提供在ApplicationContext中处理事件，如果一个bean实现ApplicationListener,那么每次ApplicationEvent被发布到ApplicationContext上，这个bean会被通知。
<!--more-->
Spring提供了以下的标准事件：

| 内置事件 | 描述 |
| ------- | ---- |
| ContextRefreshedEvent | ApplicationContext 被初始化或刷新时，该事件被发布。这也可以在 ConfigurableApplicationContext 接口中使用 refresh() 方法来发生 |
| ContextStartedEvent | 当使用 ConfigurableApplicationContext 接口中的 start() 方法启动 ApplicationContext 时，该事件被发布。你可以调查你的数据库，或者你可以在接受到这个事件后重启任何停止的应用程序 |
| ContextStoppedEvent | 当使用 ConfigurableApplicationContext 接口中的 stop() 方法停止 ApplicationContext 时，发布这个事件。你可以在接受到这个事件后做必要的清理的工作 |
| ContextClosedEvent | 当使用 ConfigurableApplicationContext 接口中的 close() 方法关闭 ApplicationContext 时，该事件被发布。一个已关闭的上下文到达生命周期末端；它不能被刷新或重启 |
| RequestHandledEvent | 这是一个 web-specific 事件，告诉所有 bean HTTP 请求已经被服务 |
由于Spring的事件处理是单线程的，所以如果一个事件被发布，直至并且除非所有接收者得到该消息，该进程被阻塞并且流程将不会继续，因此，如果事件处理被使用，在涉及应用程序时应注意。
## 监听上下文事件
为了监听上下文事件，一个bean应该实现只有一个方法onApplicationEvent() 的 ApplicationListener 接口。因此，下面提供一个例子来看看事件是如何传播的，以及如何可以用代码来执行基于某些事件所需的任务。
```java
package com.tutorialspoint;
public class HelloWorld {
   private String message;
   public void setMessage(String message){
      this.message  = message;
   }
   public void getMessage(){
      System.out.println("Your Message : " + message);
   }
}
```
下面是上下文启动事件处理类：
```java
package com.tutorialspoint;
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextStartedEvent;
public class CStartEventHandler 
   implements ApplicationListener<ContextStartedEvent>{
   public void onApplicationEvent(ContextStartedEvent event) {
      System.out.println("ContextStartedEvent Received");
   }
}
```
下面是上下文停止事件处理类：
```java
package com.tutorialspoint;
import org.springframework.context.ApplicationListener;
import org.springframework.context.event.ContextStoppedEvent;
public class CStopEventHandler 
   implements ApplicationListener<ContextStoppedEvent>{
   public void onApplicationEvent(ContextStoppedEvent event) {
      System.out.println("ContextStoppedEvent Received");
   }
}
下面试配置文件：
```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id="helloWorld" class="com.tutorialspoint.HelloWorld">
      <property name="message" value="Hello World!"/>
   </bean>

   <bean id="cStartEventHandler" 
         class="com.tutorialspoint.CStartEventHandler"/>

   <bean id="cStopEventHandler" 
         class="com.tutorialspoint.CStopEventHandler"/>

</beans>
```

```
下面在main方法中进行测试：
```java
package com.tutorialspoint;

import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MainApp {
   public static void main(String[] args) {
      ConfigurableApplicationContext context = 
      new ClassPathXmlApplicationContext("Beans.xml");

      // Let us raise a start event.
      context.start();

      HelloWorld obj = (HelloWorld) context.getBean("helloWorld");

      obj.getMessage();

      // Let us raise a stop event.
      context.stop();
   }
}
```
可以运行该应用程序,如果应用程序一切都正常，将输出以下消息
```bash
ContextStartedEvent Received
Your Message : Hello World!
ContextStoppedEvent Received
```
## 自定义事件处理
程序中往往需要自定义一些事件进行业务处理，Spring中提供了良好的事件处理机制，通过继承ApplicationEvent可以很好的实现自定义事件，下面是自定义事件的流程：
1. 通过继承 `ApplicationEvent` 创建一个事件类 `CustomEvent` ,这个类必须定义一个默认的构造函数，他应该从ApplicationEvent类中继承的构造函数。
2. 通过实现 `ApplicationEventPublisherAware` 接口创建一个事件发布者 `EventClassPublisher` ，并且还需要在XML中声明这个类作为一个bean。
3. 通过实现 `ApplicationListener` 接口创建一个事件处理者 `EventClassHandler` ，实现自定义事件的 `onApplicationEvent ` 方法，并且还需要在XML中声明这个类作为一个bean。

这个是 CustomEvent.java 文件的内容：
```java
package com.tutorialspoint;
import org.springframework.context.ApplicationEvent;
public class CustomEvent extends ApplicationEvent{ 
   public CustomEvent(Object source) {
      super(source);
   }
   public String toString(){
      return "My Custom Event";
   }
}
```
下面是 CustomEventPublisher.java 文件的内容：
```java
package com.tutorialspoint;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.context.ApplicationEventPublisherAware;
public class CustomEventPublisher 
   implements ApplicationEventPublisherAware {
   private ApplicationEventPublisher publisher;
   public void setApplicationEventPublisher
              (ApplicationEventPublisher publisher){
      this.publisher = publisher;
   }
   public void publish() {
      CustomEvent ce = new CustomEvent(this);
      publisher.publishEvent(ce);
   }
}
```
下面是 CustomEventHandler.java 文件的内容：
```java
package com.tutorialspoint;
import org.springframework.context.ApplicationListener;
public class CustomEventHandler 
   implements ApplicationListener<CustomEvent>{
   public void onApplicationEvent(CustomEvent event) {
      System.out.println(event.toString());
   }
}
```
下面是配置文件 Beans.xml：
```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id="customEventHandler" 
      class="com.tutorialspoint.CustomEventHandler"/>

   <bean id="customEventPublisher" 
      class="com.tutorialspoint.CustomEventPublisher"/>

</beans>
```
下面在main方法中进行测试：
```java
package com.tutorialspoint;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
   public static void main(String[] args) {
      ConfigurableApplicationContext context = 
      new ClassPathXmlApplicationContext("Beans.xml");    
      CustomEventPublisher cvp = 
      (CustomEventPublisher) context.getBean("customEventPublisher");
      cvp.publish();  
      cvp.publish();
   }
}
```
运行该应用程序。如果应用程序一切都正常，将输出以下信息：
```bash
My Custom Event
My Custom Event
```

