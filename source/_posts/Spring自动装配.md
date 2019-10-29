---
title: Spring自动装配
date: 2019-10-29 16:06:32
categories: Spring
tags: [Spring, 自动装配]
---
在定义bean时我们可以通过`constructor-arg`和`property`元素来注入所依赖的bean，Spring容器可以在不使用这两个标签的情况下自动装配相互协作的bean直接的关系，这有助于减少XML配置的数量。
<!--more-->
在声明bean时可以使用`autowire`元素来为一个bean定义指定自动装配模式，以下是Spring容器中的装配模式:

| 模式 | 描述 |
| -----| ---- |
| no | 默认设置，没有自动装配，应该使用显式的bean引用来进行bean的装配 |
| byname | 由属性名自动装配，Spring容器在看到XML配置中bean的自动装配属性设置为byName，然后尝试匹配，并将他的属性与配置文件中被定义为相同名称的beans的属性进行连接 |
| byType | 由属性数据类型自动装配，Spring容器在看到XML配置中bean的自动装配属性设置为byType，然后如果它的类型匹配配置文件中的一个确切的 bean 名称，它将尝试匹配和连接属性的类型。如果存在不止一个这样的 bean，则一个致命的异常将会被抛出 |
| constructor | 类似于 byType，但该类型适用于构造函数参数类型。如果在容器中没有一个构造函数参数类型的 bean，则一个致命错误将会发生 |
| autodetect | Spring首先尝试通过 constructor 使用自动装配来连接，如果它不执行，Spring 尝试通过 byType 来自动装配 |

## 自动装配的局限性
当自动装配始终在同一个项目中使用时，它的效果最好。如果通常不使用自动装配，它可能会使开发人员混淆的使用它来连接只有一个或两个 bean 定义。不过，自动装配可以显著减少需要指定的属性或构造器参数，但应该在使用它们之前考虑到自动装配的局限性和缺点。 

| 限制 | 描述 |
| ---- | ---- |
| 重写的可能性 | 可以使用总是重写自动装配的 `<constructor-arg>`和 `<property>` 设置来指定依赖关系 |
| 原始数据类型 | 不能自动装配所谓的简单类型包括基本类型，字符串和类 |
| 混乱的本质 | 自动装配不如显式装配精确，所以如果可能的话尽可能使用显式装配 |

## byName自动装配
这种模式由属性名称指定自动装配，在XML配置文件中bean的`auto-write`属性设置为byName，Spring容器将尝试将他的属性与配置文件中定义为相同名称的bean进行匹配和连接，如果找到匹配项，它将注入这些beans，否则将抛出异常。
下面是在正常情况下的配置文件 Beans.xml 文件：
```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <!-- Definition for textEditor bean -->
   <bean id="textEditor" class="com.tutorialspoint.TextEditor">
       <property name="spellChecker" ref="spellChecker" />
       <property name="name" value="Generic Text Editor" />
   </bean>

   <!-- Definition for spellChecker bean -->
   <bean id="spellChecker" class="com.tutorialspoint.SpellChecker">
   </bean>

</beans>
```
如果你要使用自动装配 “byName”，那么 XML 配置文件将成为如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <!-- Definition for textEditor bean -->
   <bean id="textEditor" class="com.tutorialspoint.TextEditor" 
      autowire="byName">
      <property name="name" value="Generic Text Editor" />
   </bean>

   <!-- Definition for spellChecker bean -->
   <bean id="spellChecker" class="com.tutorialspoint.SpellChecker">
   </bean>

</beans>
```

## byType自动装配
这种模式由属性类型指定自动装配，在XML配置文件中bean的`auto-write`属性设置为byType，Spring容器将尝试将他的属性与配置文件中定义为相同类型的bean进行匹配和连接，如果找到匹配项，它将注入这些beans，否则将抛出异常。
使用自动装配 "byType"，那么 XML 配置文件将成为如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <!-- Definition for textEditor bean -->
   <bean id="textEditor" class="com.tutorialspoint.TextEditor" 
      autowire="byType">
      <property name="name" value="Generic Text Editor" />
   </bean>

   <!-- Definition for spellChecker bean -->
   <bean id="SpellChecker" class="com.tutorialspoint.SpellChecker">
   </bean>

</beans>
```
## 由构造函数自动装配
这种模式和byType非常类似，但是它应用于构造器参数。在 XML 配置文件中 beans 的 autowire 属性设置为 constructor。然后，它尝试把它的构造函数的参数与配置文件中 beans 名称中的一个进行匹配和连线。如果找到匹配项，它会注入这些 bean，否则，它会抛出异常。

如果你要使用自动装配 “by constructor”，那么 XML 配置文件将成为如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <!-- Definition for textEditor bean -->
   <bean id="textEditor" class="com.tutorialspoint.TextEditor" 
      autowire="constructor">
      <constructor-arg value="Generic Text Editor"/>
   </bean>

   <!-- Definition for spellChecker bean -->
   <bean id="SpellChecker" class="com.tutorialspoint.SpellChecker">
   </bean>

</beans>
```





