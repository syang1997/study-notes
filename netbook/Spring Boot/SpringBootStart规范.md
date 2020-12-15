# 自动配置规范

## 一.自动配置和启动器

### 1.概念

1. Auto-configuration：自动配置负责对应用程序的当前状态做出反应并配置适当的Spring Bean。自动配置的主要驱动通常是用户的类路径。
2. Starter POMs ：负责引入通常一起使用的依赖项。

**Starter POMS配置了类路径，然后Auto-configuration对类路程做出了反应**

### 2.分离

将这两个概念分离的原因有

1. 将自动配置和依赖管理的关注点完全分开。
2. 无需Starter POMs也可以使用自动配置的优势。如使用提供依赖项的Application Server。
3. 如果依赖项由于其他原因可用，则自动配置有效。例如，如果用户已经拥有Tomcat，则无需新的启动器就可以提供特定的Tomcat支持。
4. 减少所需的启动器数量。

### 3.实现

#### 1).查找自动配置

Spring Boot检查`META-INF/spring.factories`发布的jar中是否存在文件。该文件应在`EnableAutoConfiguration`键下列出您的配置类，如以下示例所示：

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.yealink.trpc.spring.boot.starter.autoconfigure.RpcServerAutoConfiguration,\
  com.yealink.trpc.spring.boot.starter.autoconfigure.RpcClientAutoConfiguration,\
  com.yealink.trpc.spring.boot.starter.autoconfigure.RpcActuatorAutoConfiguration
```

自动配置*只能*以这种方式加载。确保在特定的程序包空间中定义了它们，并且永远不要将它们作为组件扫描的目标。

### 2).顺序配置

1. `@AutoConfigureAfter`或`@AutoConfigureBefore`
2. `@AutoConfigureOrder`

#### 2).bean的注入

##### Class Conditions

的`@ConditionalOnClass`和`@ConditionalOnMissingClass`注解让`@Configuration`类基于特定类的存在或不存在来进行配置。

##### Bean Conditions

` @ConditionalOnMissingBean`您可以使用该`value`属性按类型`name`指定bean 或按名称指定bean。

##### Prioerty Conditions

该`@ConditionalOnProperty`注解让基于Spring的环境属性配置来进行配置。使用`prefix`和`name`属性来指定应检查的属性。默认情况下，`false`匹配存在且不等于的任何属性。您还可以使用`havingValue`和`matchIfMissing`属性创建更高级的检查。

##### Resource Conditions

该`@ConditionalOnResource`注解让配置被包括仅当特定资源是否存在。例子：`file:/home/user/test.dat`。

##### Web Application Conditions

在`@ConditionalOnWebApplication`和`@ConditionalOnNotWebApplication`注释，让配置包含依赖于应用程序是否是一个“Web应用程序”。

该`@ConditionalOnWarDeployment`注解让配置取决于应用是否是被部署到一个容器中的传统WAR应用程序被包括在内。对于嵌入式服务器运行的应用程序，此条件将不匹配。

##### SpEl Expression Conditions

该`@ConditionalOnExpression`注解使用SpEL表达来确定是否配置。

### 4.规范

项目命名：

1. 不要使用`groupId`开头
2. 命名自动配置模块`acme-spring-boot`和启动器`acme-spring-boot-starter`。如果只有一个将两者结合的模块，请命名为`acme-spring-boot-starter`。

配置：

1. 多个configuration bean可以使用`@Import`关联
2. 尽量使用够构造器获取setter来设置变量，而不要使用@Autowared，@Resource等方式注入。
3. 在声明属性的时候不要使用驼峰命名法，要使用-横线分隔，这样才能支持属性名的松散规则(relaxed rules)。
4. BeanPostProcessor使用static，因为BPP在规范上要比其他bean更早注册使用，接口里方法在调用的时候（创建Transaction相关的Bean的时候）可以直接使用这个static实例，而不要等到这个Configuration类的其他的Bean都构建好。

配置文件：

1. 请勿以“ The”或“ A”开头描述。
2. 对于`boolean`类型，使用 "Whether" or "Enable"来描述。
3. 对于基于集合的类型，请以“Comma-separated list”开始描述。
4. 如果默认单位与毫秒不同，请使用`java.time.Duration`而不是`long`并描述默认单位，例如“如果未指定持续时间后缀，将使用秒”。
5. 除非必须在运行时确定默认值，否则请不要在描述中提供默认值。

依赖：

1. 不要在启动器POM中放置任何代码
2. 不要在同一启动程序中尝试支持几代Spring Boot（即Spring Boot 1和Spring Boot 2）
3. 不要创建Spring Boot已经涵盖的入门POM。例如，不要`my-project-starter-tomcat`在`my-project-autoconfigure`+ `spring-boot-starter-web`足够的情况下启动。

