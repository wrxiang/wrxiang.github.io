---
title: Spring IOC容器
date: 2019-10-28 15:17:26
categories: Spring
tags: [Spring, IOC]
---
Spring容器是Spring框架的核心，容器创建配置并创建对象，并管理它们的生命周期（从创建到销毁），Spring容器通过依赖注入管理整个应用程序中的对象，这些对象被称为Spring Beans。
通过读取配置元数据提供的指令，容器知道需要对哪些对象进行实例化、配置以及组装。配置元数据可以通过XML、Java注解或Java代码来完成。IOC容器是具有依赖注入功能的容器，它可以配置并创建管理对象，IOC容器负责实例化、定位、配置应用程序中的对象以及建立这些对象间的依赖。通常new一个实例由程序员来控制，而现在这个过程交给Spring容器来完成，这就是常说的“控制反转”。
<!--more-->
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

**singleton 作用域**：singleton是默认的作用域，当定义bean时如果没有指定作用域配置项，则这个bean的作用域被默认为singleton。当一个bean的作用域被定义为singleton，那么Spring IOC容器中只会存在一个共享的bean实例，也就是说，只会创建该bean定义的唯一实例。
singleton是单例类型，在创建容器时就同事自动创建一个bean对象，不管是否使用，它都存在，而且每次获取到的对象都是同一个。
**prototype 作用域**：当一个bean的作用域被定义为prototype，表示一个bean定义对应多个对象实例，prototype作用域的bean会导致在每次对该bean的请求（将其注入另一个bean或者以代码的方式调用容器中的getBean()方法）时都会创建一个新的bean实例。
prototype是原型类型，它在我们创建容器的时候并没有实例化，而是当获取bean时才会创建对象，而且每次获取到的对象都不是同一个对象。

## Spring Bean生命周期
Spring Bean的生命周期可以表示为：bean的定义->bean的初始化->bean的使用->bean的销毁，当一个bean被实例化时，它可能需要执行一些初始化操作使它转换成可用的状态，同样，当bean不再需要，并且要从容器中移除时，可能需要做一些清除工作。我们在定义bean时，声明两个属性： **init-method** 和  **destroy-method**，init-method属性指定一个方法，在实例化bean时，立即调用该方法，同样，destroy-method指定一个方法，只有从容器中移除bean时才调用该方法。
```XML
<bean id="exampleBean" 
         class="examples.ExampleBean" init-method="init" destroy-method="destroy"/>
```
如果程序中有太多相同名称的初始化或销毁方法的bean，那么我们不需要在每一个bean上声明初始化或销毁方法。框架使用元素中的**default-init-method** 和 **default-destroy-method** 属性提供了灵活地配置这种情况，如下所示：
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd"
    default-init-method="init" 
    default-destroy-method="destroy">

   <bean id="..." class="...">
       <!-- collaborators and configuration for this bean go here -->
   </bean>

</beans>
```
## Spring Bean继承
bean中可以包含很多的配置信息，包括构造函数的参数，属性值，容器的具体信息，例如初始化方法，静态工厂方法等，在Spring中可以定义继承父定义的配置数据，子定义可以根据需要重写一些值或者添加其他值。
Spring Bean定义的继承和Java类的继承无关，但是继承的概念是一样的，可以定义一个父bean作为模板，其他子bean就可以从父bean中继承所需要的配置。
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id="helloWorld" class="com.tutorialspoint.HelloWorld">
      <property name="message1" value="Hello World!"/>
      <property name="message2" value="Hello Second World!"/>
   </bean>

   <bean id="helloIndia" class="com.tutorialspoint.HelloIndia" parent="helloWorld">
      <property name="message1" value="Hello India!"/>
      <property name="message3" value="Namaste India!"/>
   </bean>

</beans>
```

## Spring Bean后置处理器
Spring Bean后置处理器允许在调用初始化方法前后对bean进行额外的处理，**BeanPostProcessor** 接口定义回调方法，可以实现该方法来提供自己的实例化逻辑，依赖解析逻辑等，也可以在 Spring 容器通过插入一个或多个 BeanPostProcessor 的实现来完成实例化，配置和初始化一个bean之后实现一些自定义逻辑回调方法。
可以配置多个 BeanPostProcessor 接口，通过设置 BeanPostProcessor 实现的 Ordered 接口提供的 order 属性来控制这些 BeanPostProcessor 接口的执行顺序。
BeanPostProcessor 可以对 bean（或对象）实例进行操作，这意味着 Spring IoC 容器实例化一个 bean 实例，然后 BeanPostProcessor 接口进行它们的工作。 ApplicationContext 会自动检测由 BeanPostProcessor 接口的实现定义的 bean，注册这些 bean 为后置处理器，然后通过在容器中创建 bean，在适当的时候调用它。
```Java
package com.tutorialspoint;
public class HelloWorld {
   private String message;
   public void setMessage(String message){
      this.message  = message;
   }
   public void getMessage(){
      System.out.println("Your Message : " + message);
   }
   public void init(){
      System.out.println("Bean is going through init.");
   }
   public void destroy(){
      System.out.println("Bean will destroy now.");
   }
}
```
这是实现 BeanPostProcessor 的非常简单的例子，它在任何 bean 的初始化的之前和之后输入该 bean 的名称。可以在初始化 bean 的之前和之后实现更复杂的逻辑，因为有两个访问内置 bean 对象的后置处理程序的方法。 
```Java
package com.tutorialspoint;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.beans.BeansException;
public class InitHelloWorld implements BeanPostProcessor {
   public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
      System.out.println("BeforeInitialization : " + beanName);
      return bean;  // you can return any other object as well
   }
   public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
      System.out.println("AfterInitialization : " + beanName);
      return bean;  // you can return any other object as well
   }
}
```
下面是 MainApp.java 文件的内容。在这里，你需要注册一个在 AbstractApplicationContext 类中声明的关闭 hook 的 registerShutdownHook() 方法。它将确保正常关闭，并且调用相关的 destroy 方法。
```Java
package com.tutorialspoint;
import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
   public static void main(String[] args) {
      AbstractApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");
      HelloWorld obj = (HelloWorld) context.getBean("helloWorld");
      obj.getMessage();
      context.registerShutdownHook();
   }
}
```
下面是 init 和 destroy 方法需要的配置文件 Beans.xml 文件：
```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id="helloWorld" class="com.tutorialspoint.HelloWorld"
       init-method="init" destroy-method="destroy">
       <property name="message" value="Hello World!"/>
   </bean>

   <bean class="com.tutorialspoint.InitHelloWorld" />
</beans>
```
一旦你创建源代码和 bean 配置文件完成后，我们就可以运行该应用程序。如果你的应用程序一切都正常，将输出以下信息：
```base
BeforeInitialization : helloWorld
Bean is going through init.
AfterInitialization : helloWorld
Your Message : Hello World!
Bean will destroy now.
```








