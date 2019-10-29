---
title: Spring IOC容器
date: 2019-10-28 15:17:26
categories: Spring
tags: [Spring, IOC]
---
Spring容器是Spring框架的核心，容器创建配置并创建对象，并管理它们的生命周期（从创建到销毁），Spring容器通过依赖注入管理整个应用程序中的对象，这些对象被称为Spring Beans。
通过读取配置元数据提供的指令，容器知道需要对哪些对象进行实例化、配置以及组装。配置元数据可以通过XML、Java注解或Java代码来完成。IOC容器是具有依赖注入功能的容器，它可以配置并创建管理对象，IOC容器负责实例化、定位、配置应用程序中的对象以及建立这些对象间的依赖。通常new一个实例由程序员来控制，而现在这个过程交给Spring容器来完成，这就是常说的“控制反转”。
## Spring BeanFactory容器
Spring BeanFactory是最简单一个容器，它主要的功能是为依赖注入提供支持，这个容器接口在`org.springframework.beans.factory.BeanFactor`中被定义。BeanFactory和相关的接口，比如`BeanFactoryAware、DisposableBean、InitializingBean`，仍保留在Spring中，主要的目的是向后兼容已存在的那些和Spring整合在一起的第三方框架。
在Spring中，有大量对BeanFactory接口的实现，其中最常使用的是`XMLBeanFactory`类，这个容器从一个xml文件中读取配置元数据，由这些元数据来生成一个被配置化的系统或者应用。
## Spring ApplicationContext 容器
ApplicationContext是Spring中比较高级的容器，和BeanFactory类似，它可以加载配置文件中定义的bean，并维护它们之间的依赖关系，另外，它增加了企业级开发中所需要的功能，比如从属性文件中解析文本信息和将事件传递给所指定的监听器。这个容器在`org.springframework.context.ApplicationContext interface`接口中定义，ApplicationContext包含BeanFactory中所有功能，一般情况下，ApplicationContext会更加优秀，但是BeanFactory仍在一些轻量级的应用中使用，比如移动设备或者基于applet的应用程序。
常用的ApplicationContext 接口实现：
* **FileSystemXmlApplicationContext**：该容器从XML文件中加载已被定义的bean，这里需要提供给构造器XML文件的完整路径
* **ClassPathXmlApplicationContext**：该容器从 XML 文件中加载已被定义的 bean。在这里，你不需要提供 XML 文件的完整路径，只需正确配置 CLASSPATH 环境变量即可，因为，容器会从 CLASSPATH 中搜索 bean 配置文件。
* **WebXmlApplicationContext**：该容器会在一个 web 应用程序的范围内加载在 XML 文件中已被定义的 bean。

## Spring bean定义
bean是一个被实例化，组装并通过Spring IOC容器所管理的对象，这些对象是由用容器提供的配置元数据创建的。
bean常用配置属性如下：

| 属性 |   描述  | 
| --- | ------ |
| class | 这个属性是强制性的，并且指定用来创建 bean 的 bean 类 | 
| name | 这个属性指定唯一的 bean 标识符。在基于 XML 的配置元数据中，你可以使用 ID 和/或 name 属性来指定 bean 标识符 |
| scope | 这个属性指定由特定的 bean 定义创建的对象的作用域，默认是单例模式  |
| constructor-arg | 它是用来注入依赖关系的 |
| properties | 它是用来注入依赖关系的 |
| autowiring mode | 它是用来注入依赖关系的 | 
| lazy-initialization mode | 延迟初始化的 bean 告诉 IoC 容器在它第一次被请求时，而不是在启动时去创建一个 bean 实例 |
| initialization | 在 bean 的所有必需的属性被容器设置之后，调用回调方法 |
| destruction | 当包含该 bean 的容器被销毁时，使用回调方法 |

## Spring bean作用域
当在Spring定义一个bean时，必须声明其作用域，Spring中支持五个作用域：`singleton、prototype、request、session和global session`，以下是五种作用域的说明：

| 作用域 | 描述    | 
| ----- | ------- |
| singleton | 在spring IoC容器仅存在一个Bean实例，Bean以单例方式存在，默认值 |
| prototype | 每次从容器中调用Bean时，都返回一个新的实例，即每次调用getBean()时，相当于执行newXxxBean()  |
| request | 每次HTTP请求都会创建一个新的Bean，该作用域仅适用于WebApplicationContext环境 |
| session | 同一个HTTP Session共享一个Bean，不同Session使用不同的Bean，仅适用于WebApplicationContext环境 |
| global-session | 一般用于Portlet应用环境，该运用域仅适用于WebApplicationContext环境 |





