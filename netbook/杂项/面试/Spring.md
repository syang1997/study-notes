# Sprin

## 1.IOC 和DI的区别

控制反转（Inversion of Control，缩写为IoC），是面向对象编程中的一种设计原则，可以用来减低计算机代码之间的耦合度。其中最常见的方式叫做依赖注入（Dependency Injection，简称**DI**），还有一种方式叫“依赖查找”（Dependency **Lookup**）

也就是说，IOC是目标，DI是实现手段。

## 2.spring实现IOC的思路和方法

spring实现IOC的思路是提供一些配置信息用来描述类之间的依赖关系，然后由容器去解析这些配置信息，继而维护好对象之间的依赖关系，前提是对象之间的依赖关系必须在类中定义好，比如A.class中有一个B.class的属性，那么我们可以理解为A依赖了B。既然我们在类中已经定义了他们之间的依赖关系那么为什么还需要在配置文件中去描述和定义呢？

spring实现IOC的思路大致可以拆分成3点

1. 应用程序中提供类，提供依赖关系（属性或者构造方法）

1. 把需要交给容器管理的对象通过配置信息告诉容器（xml、annotation，javaconfig）

1. 把各个类之间的依赖关系通过配置信息告诉容器



配置这些信息的方法有三种分别是xml，annotation和javaconfig

维护的过程称为自动注入，自动注入的方法有两种构造方法和setter

自动注入的值可以是对象，数组，map，list和常量比如字符串整形等

### BeanFactory 的生命流程

上面简述了 toy-spring 项目的编码背景，接下来，在本节中，我将向大家介绍 toy-spring 项目中 IOC 部分的实现原理。在详细介绍 IOC 的实现原理前，这里先简单说一下 BeanFactory 的生命流程：

1. BeanFactory 加载 Bean 配置文件，将读到的 Bean 配置封装成 BeanDefinition 对象
2. 将封装好的 BeanDefinition 对象注册到 BeanDefinition 容器中
3. 注册 BeanPostProcessor 相关实现类到 BeanPostProcessor 容器中
4. BeanFactory 进入就绪状态
5. 外部调用 BeanFactory 的 getBean(String name) 方法，BeanFactory 着手实例化相应的 bean
6. 重复步骤 3 和 4，直至程序退出，BeanFactory 被销毁

## 3.Aop是什么

与OOP对比，面向切面，传统的OOP开发中的代码逻辑是自上而下的，而这些过程会产生一些横切性问题，这些横切性的问题和我们的主业务逻辑关系不大，这些横切性问题不会影响到主逻辑实现的，但是会散落到代码的各个部分，难以维护。AOP是处理一些横切性问题，AOP的编程思想就是把这些问题和主业务逻辑分开，达到与主业务逻辑解耦的目的。使代码的重用性和开发效率更高。

https://shimo.im/docs/Nj0bcFUy3SYyYnbI/read

## 4.动态代理底层技术

|                                      | JDK动态代理    | CGLIB代理      |
| ------------------------------------ | -------------- | -------------- |
| 编译时期的织入还是运行时期的织入?    | 运行时期织入   | 运行时期织入   |
| 初始化时期织入还是获取对象时期织入？ | 初始化时期织入 | 初始化时期织入 |

jdk

* 类 java. lang reflect Proxy

* 接口 vocation handler

* 只能基于接口进行动态代理

https://www.javadoop.com/post/spring-ioc

