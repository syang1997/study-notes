# Spring5的日志

**Spring4的日志与Spring5不同,Spring4是使用的log4j,而Spring5并不是;**

## 一主流的日志

### 1.log4j

<!--<dependency>-->

​    <!--<groupId>log4j</groupId>-->

​    <!--<artifactId>log4j</artifactId>-->

​    <!--<version>1.2.12</version>-->

<!--</dependency>-->

可以不需要依赖第三方的技术

直接记录日志

### 2.jcl

jakartaCommonsLoggingImpl

<**dependency**>

​    <**groupId**>commons-logging</**groupId**>

​    <**artifactId**>commons-logging</**artifactId**>

​    <**version**>1.1.1</**version**>

</**dependency**>

jcl他不直接记录日志,他是通过第三方记录日志(jul)



如果使用jcl来记录日志,在没有log4j的依赖情况下,是用jul

如果有了log4j则使用log4j

**jcl**=Jakarta commons-logging ,是apache公司开发的一个抽象日志通用框架,本身不实现日志记录,但是提供了记录日志的抽象方法即接口(info,debug,error.......),底层通过一个数组存放具体的日志框架的类名,然后循环数组依次去匹配这些类名是否在app中被依赖了,如果找到被依赖的则直接使用,所以他有先后顺序。

![img](https://images-cdn.shimo.im/yFBaGU5euKMuCqtu/jcl1.png!thumbnail)



上图为**jcl中存放日志技术类名的数组，默认有四个，后面两个可以忽略。**

![img](https://images-cdn.shimo.im/WUYYvMBBkx4eL28s/jcl0.png!thumbnail)

上图81行就是通过一个类名去load一个**class**，如果**load**成功则直接**new**出来并且返回使用。

如果没有load到**class**这循环第二个，直到找到为止。

![img](https://images-cdn.shimo.im/8MzEr8n4FycGtto5/jcl2.png!thumbnail)

可以看到这里的循环条件必须满足result不为空，

也就是如果没有找到具体的日志依赖则继续循环，如果找到则条件不成立，不进行循环了。

总结：顺序log4j>jul



### 3.jul

java自带的一个日志记录的技术,直接使用

### 4.log4j2

### 5.**slf4j**

**slf4j他也不记录日志,通过绑定器绑定一个具体的日志记录来完成日志记录**

log4j---<**artifactId**>slf4j-log4j12</**artifactId**>

### 6.logback

### 7.simple-log



















