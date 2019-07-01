---
title:  Spring 解析
date: 2019-03-31 09:19:58
tags:
---

###  Spring 解析

<!--more-->



#### IOC

从代码角度来看,IOC容器不过是Spring中定义的具有IOC基本功能的一些类的统称,这些类都遵循一些共同的接口规范,所以我们可以说实现某些接口的具体的实现类就是IOC容器。而IOC容器的启动流程,说白了就是创建并初始化一个该实现类的实例的过程,在这个过程中要进行诸如配置文件的加载解析,核心组件的注册,bean 实例的创建等一系列繁琐复杂的操作,因而整个过程显得相对漫长,逻辑也相对复杂。

Spring 容器类可以分为两大类：

1. 一类是由BeanFactory接口定义的核心容器。BeanFactory位于整个容器类体系结构的顶端,其基本实现类为DefaultListableBeanFactory。之所以称其为核心容器,是因为该类容器实现IOC的核心功能:比如配置文件的加载解析,Bean依赖的注入以及生命周期的管理等。BeanFactory作为Spring框架的基础设施,面向Spring框架本身,一般不会被用户直接使用。
2. 另一类则是由ApplicationContext接口定义的容器,通常译为应用上下文,不过称其为应用容器可能更形象些。它在BeanFactory提供的核心IOC功能之上作了扩展。通常ApplicationContext的实现类内部都持有一个BeanFactory的实例,IOC容器的核心功能会交由它去完成。而ApplicationContext本身,则专注于在应用层对BeanFactory作扩展,比如提供对国际化的支持,支持框架级的事件监听机制以及增加了很多对应用环境的适配等。ApplicationContext面向的是使用Spring框架的开发者。开发中经常使用的ClassPathXmlApplicationContext就是典型的Spring的应用容器,也是要进行解读的IOC容器。



在web.xml 中会有

```xml
<context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
```

代表了web容器启动时首先进入ContextLoaderListener 这个类，并且之后回去加载classpath下的applicationContext.xml文件。     初始化XmlBeanDefinitionReader,同时初始化了

registry（BeanDefineRegister）：该类的作用主要是注册 BeanDefinition 实例。

resourceLoader（ResourceLoader）：该类来用来查找和定位资源。

environment（Environment）：表示当前应用程序正在运行的环境。



BeanFactory ：

- XmlBeanFactory 通过 Resource 装载 Spring 配置信息并启动 IoC 容器，然后就可以通过 BeanFactory#getBean(beanName)方法从 IoC 容器中获取 Bean 了。通过 BeanFactory 启动IoC 容器时，并不会初始化配置文件中定义的 Bean，初始化动作发生在第一个调用时。
- 对于单实例（ singleton）的 Bean 来说，BeanFactory会缓存 Bean 实例，所以第二次使用 getBean() 获取 Bean 时将直接从 IoC 容器的缓存中获取 Bean 实例。Spring 在 DefaultSingletonBeanRegistry 类中提供了一个用于缓存单实例 Bean 的缓存器，它是一个用HashMap 实现的缓存器，单实例的 Bean 以 beanName 为键保存在这个HashMap 中。

ApplicationContext ： 从BeanFactory 派生，面向开发者，可以通过配置的方式实现

- １、ResourceLoader从存储介质中加载Spring配置信息，并使用Resource表示这个配置文件的资源；
- ２、BeanDefinitionReader读取Resource所指向的配置文件资源，然后解析配置文件。配置文件中每一个<bean>解析成一个BeanDefinition对象，并保存到BeanDefinitionRegistry中；
- ３、容器扫描BeanDefinitionRegistry中的BeanDefinition，使用Java的反射机制自动识别出Bean工厂后处理后器（实现BeanFactoryPostProcessor接口）的Bean，然后调用这些Bean工厂后处理器对BeanDefinitionRegistry中的BeanDefinition进行加工处理。主要完成以下两项工作：
- - 1）对使用到占位符的<bean>元素标签进行解析，得到最终的配置值，这意味对一些半成品式的BeanDefinition对象进行加工处理并得到成品的BeanDefinition对象；
  - 2）对BeanDefinitionRegistry中的BeanDefinition进行扫描，通过Java反射机制找出所有属性编辑器的Bean（实现java.beans.PropertyEditor接口的Bean），并自动将它们注册到Spring容器的属性编辑器注册表中（PropertyEditorRegistry）；
- 4．Spring容器从BeanDefinitionRegistry中取出加工后的BeanDefinition，并调用InstantiationStrategy着手进行Bean实例化的工作；
- 5．在实例化Bean时，Spring容器使用BeanWrapper对Bean进行封装，BeanWrapper提供了很多以Java反射机制操作Bean的方法，它将结合该Bean的BeanDefinition以及容器中属性编辑器，完成Bean属性的设置工作；
- 6．利用容器中注册的Bean后处理器（实现BeanPostProcessor接口的Bean）对已经完成属性设置工作的Bean进行后续加工，