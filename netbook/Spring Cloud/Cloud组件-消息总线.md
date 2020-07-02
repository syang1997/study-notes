# Cloud组件-消息总线

## 一.SpringCloud Bus

### 1.简介

Spring Cloud Bus配合 Spring Cloud Config使用可以实现配置的动态刷新。

Spring Cloud Bus是用来将分布式系统的节点与轻量级消息系统链接起来的框架，它整合了Java的事件处理机制和消息中间件的功能Spring Clud Bus目前支持 RabbitMQ和 Kafka。

什么是总线

在微服务架构的系统中，通常会使用轻量级的消息代理来构建_个共用的消息主题，并让系统中所有微服务实例都连接上来。由于该主题中产生的消息会被所有实例监听和消费，所以称它为消息总线。在总线上的各个实例，都可以方便地广播一些需要让其他连接在主题上的实例都知道的消息

基本原理

ConfigClient实例都监听MQ中同—个 topIc（默认是 springCloudBus）。当一个服务刷新数据的时候，它会把这个信息放入到 Topic中，这样其它监听同一 Topick的服务就能得到通知，然后去更新自身的配置。

### 2.整合RabbitMq使用

#### 1).center端

在center端中依赖

```xml
        <!--消息总线对rabbitmq的支持 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
```

在yml配置

```yaml
rabbitmq:
  host: 118.24.123.205
  port: 5672
  username: guest
  password: guest

management:
  endpoints:
    web:
      exposure:
        include: "bus-refresh"  #暴露bus配置的端点
```

#### 2).client端

在center端中依赖

```xml
        <!--消息总线对rabbitmq的支持 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
```

在yml配置

```yaml
spring:
  rabbitmq:
    host: 118.24.123.205
    port: 5672
    username: guest
    password: guest

management:
  endpoints:
    web:
      exposure:
        include: "*"  #暴露bus配置的端点
```

#### 2).通知

在修改完配置之后，通过`http://localhost:3344/actuator/bus-refresh`请求对center进行

