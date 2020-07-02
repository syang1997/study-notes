# Cloud组件-服务调用

## 一.Ribbon

### 1.简介

Spring Cloud Ribbon是基于 Netflⅸ Ribbon实现的一套==客户端==负载均衡的工具。简单的说， Ribbon是Netflⅸ发布的开源项目，主要功能是提供客户端的软件==负载均衡算法和服务调用==。 Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，就是在配置文件中列岀 Load balancer（简称LB）后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器。我们很容易使Ribbonn实现自定义的负载均衡算法。

#### 1).特点

负载均衡LB

* 集中式

  即在服务的消费方和提供方之间使用独立的LB设施可以是硬件，如F5，也可以是软件，如 nginx，由该设施负责把访问请求通过某种策略转发至服务的提供方；

* 进程式

  将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器。
  Ribbon就属于进程内LB，它只是个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址

### 2.整合SpringCloud

在pom中引入依赖，新版的Eureka依赖中已经集成了Ribbon，可以直接使用。

### 3.IRule

lRule：根据特定算法中从服务列表中选取一个要访问的服务

1. RoundRobinRule：轮询
2. RandomRule：随机
3. RetryRule：先按照轮询，如果失败则在指定时间内重试
4. WeightedResponseTimeRuleL：对随机的扩展，响应速度越快权重越大，越容易被选择
5. BestAvailableRule：会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务
6. Availability Filtering：Rule先过滤掉故障实例，再选择并发较小的实例
7. ZoneAvoidancerule：默认规则，复合判断 server所在区域的性能和 server的可用性选择服务器

#### 1).替换

这个自定义配置类不能放在@ ComponentScan，所扫描的当前包下以及子包下否则我们自定义的这个配置类就会被所有的 Ribbon客户端所共享，达不到特殊化定制的目的了。

由于@SpringBootApplicatio带有ComponetScan所以要在创建新的文件路径。

1. 创建package
2. 创建MySelfRule

```java
@Configuration
public class MySelfRule {
    @Bean
    public IRule myRules(){
        //定义为随机规则
        return new RandomRule();
    }
}
```

3. 在主启动类中添加@RibbonClient

```java
@SpringBootApplication
@EnableEurekaClient
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration = MySelfRule.class)
public class OrderMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain80.class,args);
    }
}
```

## 二.OpenFeign

### 1.简介

Feign是一个声明式 WebService客户端。使用 Feign能让编写 Web Service客户端更加简单。它的使用方法是定义一个服务接口然后在上面添加注解。 Feign也支持可拔插式的编码器和解码器。 Spring cloud对 Feign进行了封装，使其支持了 Spring MVC标准注解和 HttpmessagecoNverters. Feign可以与 Eureka和 Ribbonη组合使用以支持负载均衡

#### 1).特点

Feign旨在使编写 Java Http客户端变得更容易。前面在使用 Ribbon+ RestTemplate时，利用 RestTemplate对htt请求的封装处理，形成了一套模版化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，往往一个接口会被多处调用，所以通常都会针对毎个微服务自行封装一些客户端类来包装这些依赖服务的调用。所以， Feignη在此基础上做了进一步封装，由他来帮助我们定义和实现依赖服务接口的定义。在 Feign的实现下，我们只需创建一个接口并使用注解的方式来配置它（以前是Dao接口上面标注 Mappe注解现在是一个微服务接口上面标注一个Feign注解即可，即可完成对服务提供方的接囗绑定，简化了使用 Spring cloud Ribbon时，自动封装服务调用客户端的开发量。

利用 Ribbon维护了 Payment的服务列表信息，并且通过轮询实现了客户端的负载均衡。而与 Ribbon不同的是，通过 feign只需要定义服务绑定接口且以声明式的方法，优雅而简单的实现了服务调用

#### 2).区别

| Fegin                                                        | OpenFegin                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Feign是SpringCloud组件中的个轻量级RESTfu的HTTP服务客户端Feign内置了 Ribbon，用来做客户端负载均衡，去调用服务注册中心的服务。 Feign的使用方式是：使用 Feign的注解定义接口，调用这个接口就可以调用服务注册中心的服务 | Open Feign是 Spring Cloud在 Feign的基础上支持了 SpringMVO的注解，如@ RequesMapping等等。 Open Feign的@ FeignClient可以解析SpringMVC的@ RequestMapping注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务。 |
| <dependency><br/><groupId>org. springframework cloud</groupId><br/><artifactId>spring-cloud-starter-feign</artifactId><br/></dependency> | <dependency><br/><groupId>org. springframework cloud</groupId><br/><artifactId>spring-cloud-starter-openfeign</artifactId><br/></dependency> |

### 2.整合SpringCloud

#### 1).使用步骤

1. 在pom中添加openfeign的依赖
2. 在启动类上添加注解`@EnableFeignClients`
3. 创建调用服务的接口，在接口中使用注解`@FeignClient(value = "CLOUD-PAYMENT-SERVICE")`
4. 构建一个controller层调用feignservice

#### 2).超时控制

默认Feigη客户端只等待秒钟，但是服务端处理需要超过1秒钟，导致 Feign客户端不想等待了，直接返回报错。为了避免这样的情况，有时候我们需要设置 Feign客户端的超时控制。

```yml
ribbon:
  #并指的是建立连接的时问，适用于网络状况正常的情况下，两端连接所用的时间
  ReadTimeout: 50000
  #指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 50000
```







