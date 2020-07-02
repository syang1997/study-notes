# Cloud组件-服务配置

## 一.SpringCloudConfig

### 1.简介

微服务意味着要将单体应用中的业务拆分成一个个子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行，所以一套集中式的、动态的配置管理设施是必不可少的。
Spring Cloud提供了 Config Server来解决这个问题，我们每个微服务自己带着个 application. yml，上百个配置文件的管理

Spring Cloud Config为微服务架构中的微服务提供集中化的外部配置攴持，配置服务器为==各个不同微服务应用==的所有环境提供了—个==中心化的外部配置==。

Spring Cloud Config分为==服务端和客户端==两部分，服务端也称为分布式配置中心，它是个==独立==的微服务应用，用来连接配置服务器并达客户端提供获取配置信息，加密/解密信息等访问接口客户端则是通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息配置服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过gt客户端工具来方便的管理和访问配置内容

### 2.特性

1. 集中管理配置文件
2. 不同环境不同配置，动态化的配置更新，分环境部署比如dev/ /test/prod/beta/ release
3. 运行期间动态调整配置，不再需要在每个服务部罟的机器上编写配置文件，服务会向配置中心统一拉取配置自己的信息
4. 当配置发生变动时，服务不需要重启即可感知到配置的变化并应用新的配置
5. 将配置信息以REST接口的形式暴露

### 3.使用

#### 1).server端

1. pom

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
```

2. yml

```yaml
server:
  port: 3344
spring:
  profiles:
    active: git
  application:
    name: cloud-config-center
  cloud:
    config:
      server:
        git:
          uri: https://github.com/syang1997/SpringCloud-Config.git
          #搜索目录
          search-paths:
            - SpringCloud-Config
          username: **********
          password: **********
      #读取分支
      label: master

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/

```

3. 主启动`@EnableConfigServer`

4. 访问规则

* ***\*/{application}-{profile}.yml\****
* ***\*/{application}/{profile}[/{label}]\****
* ***\*/label/{application}-{profile}.yml\****

#### 2).client端

applicaiton.ym是用户级的资源配置项

bootstrap.ym是系统级的，优先级更加高

Spring Cloud会创建个“ Bootstrap Context"，作为 Spring应用的 Application Context的父上下文。初始化的时候， Bootstrap Context负责从外部源加载配置属性并解析配置。这两个上下文共享—个从外部获取的 Environment。

Bootstrap`属性有高优先级，默认情况下，它们不会被本地配置覆盖。 Bootstrap coηtext和 Application Context有着不同的约定，所以新增了一个 bootstrap ym文件，保证 Bootstrap Context和 Application Context配置的分离。

要将cent模块下的 application.ym文件改为 bootstrap ym这是很关键的，因为 bootstrap yml是比 application. ym先加载的。 bootstrap ym优先级高于 application. yml

1. pom

```yaml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
```

2. yml

bootstrap.yml

```yml
server:
  port: 3355

spring:
  application:
    name: config-client             			#对应微服务配置文件名称
  cloud:
    config:
      name: config
      uri: http://localhost:3344/    		        #config server 端地址
      profile: dev                                      #项目配置文件选择
      label: master
eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/
```



3. rest接口业务类

```java
@RestController
@RefreshScope
public class ConfigController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo(){
        return configInfo;
    }
}

```







