# Cloud-服务降级

## 一.Hystrix

### 1.概述

#### 1).分布式系统的问题

复杂分布式体系结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免地失败。

#### 2).服务雪崩

多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其它的微服务，这就是所谓的出”。

如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的“雪崩效应”。

对于高流量的应用来说，单的后端依赖可能会导致所有服务器上的所有资源都在几秒钟内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障。这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统。

通常当你发现一个模块下的某个实例失败后，这时候这个模块依然还会接收流量，然后这个有问题的模块还调用了其他的模块，这样就会发生级联故障，或者叫雪崩。

### 2.简介

Hystrⅸ是一个用于处理分布式系统的==延迟==和==容错==的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等， Hystrix能够保证在—个依赖出问题的情况下，==不会导致整体服务失败，避免級联故障，以提高分布式系统的殚弹性。==
断路器”本身是_种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），==向调用方返回一个符合预期的、可处理的备选响应（ FallBack），而不是长时间的等待或者抛出调用方无法处理的异常==，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

### 3.重要概念

#### 1).服务降级：

服务器忙，请稍后再试，不让客户端等待并立刻返回一个友好提示， fallback

哪些情况会出发降级:

1. 程序运行异常
2. 超时
3. 服务熔断触发服务降级
4. 线程池/信号量打满也会导致服务降级

#### 2).服务熔断

类比保险丝达到最大服务访问后，直接拒绝访问，拉闸限电，然后调用服务降级的方法并返回友好提示

熔断机制是应对雪崩效应的种微服务链路保护机制。当扇出链路的某个微服务出错不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息==当检测到该节点微服务调用响应正常后，恢复调用链路。==

#### 3).服务限流

秒杀高并发等操作，严禁一窝蜂的过来拥挤，大家排队，一秒钟N个，有序进行

### 4.整合使用服务降级

#### 1).在pom引入依赖

```xml
        <!--hystrix-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
```

#### 2).主启动类

在主启动类中添加`@EnableCircuitBreaker`

#### 3).注解@HystrixCommand

```java
@HystrixCommand(fallbackMethod = "paymentTimeOutHandler",commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "3000")
    })
    public String paymentTimeOut(Integer id){
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "线程池"+Thread.currentThread().getName()+"TimeOut"+id;
    }

    public String paymentTimeOutHandler(Integer id){
        return "线程池"+Thread.currentThread().getName()+"TimeOut"+id+"失败请重试";
    }
```

在业务类中需要进行降级的方法上添加注解`@HystrixCommand`在出现`@HystrixProperty`中的情况或者服务不可用时，例如上述的响应时间超过3s时，会调用fallbackMethod方法。

#### 4).全局配置

在业务类上添加`@DefaultProperties(defaultFallback = "paymentGlobalFallbackMethod")`，来指定默认的fallbackMethod方法。来降低代码膨胀。

#### 5).在消费端中

如果在消费端中也使用熔断：

1. 在yml中配置feign.hystrix.enabled: true
2. 主启动中`@EnableHystrix`
3. 业务类中配置`@HystrixCommand`

#### 6).在fegin中配置统一的fallback

实习fegin的接口

```java
@Component
public class PaymentFallbackService implements PaymentFeginService{
    @Override
    public String paymentok(Integer id) {
        return "----ok----";
    }

    @Override
    public String paymentTimeOut(Integer id) {
        return "******to*****";
    }
}
```

在fegin接口中的注解`@FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT",fallback = PaymentFallbackService.class)`指定上实习接口的fallback类。

### 5.整合使用服务熔断

在 Spring Cloud框架里，熔断机制通过 Hystrix实现。 Hystrⅸ会监控微服务间调用的状况，当失败的调用到定阈值，缺省是5秒内20次调用失败，就会启动熔断机制。熔断机制的注解是`@HystrixCommand`

```java
@HystrixCommand(fallbackMethod = "paymentCircuitBreakerFallback",commandProperties = {
            @HystrixProperty(name = "circuitBreak.enable",value ="true"),//是否开启断路器
            @HystrixProperty(name = "circuitBreak.requestVolumeThreshold",value ="10"),//请求次数
            @HystrixProperty(name = "circuitBreak.sleepWindowInMilliseconds",value ="10000"),//时间窗口期
            @HystrixProperty(name = "circuitBreak.errorThresholdPercentage",value ="60")//失败率达到多少后跳闸
    })
```

熔断类型

1. 熔断打开：请求不再进行调用当前服务，内部设置时钟一般为MTTR（平均故障处理时间），当打开时长达到所设时钟则进入半熔断状态
2. 熔断关闭：熔断关闭不会对服务进行熔断
3. 熔断半开：部分请求根据规则调用当前服务，如果请求成功且符合规则则认为当前服务恢复正常，关闭熔断

涉及到断路器的三个重要参数：快照时间窗、请求总数阀值、错误百分比阀值。

1. 快照时间窗：断路器确定是否打开需要统计些请求和错误数据，而统计的时间范围就是快照时间窗，默认为最近的10秒。
2. 请求总数阀值：在快照时间窗内，必须满足请求总数阀值才有资格熔断。默认为20，意味着在10秒内，如果该 hystⅸx命令的调用次数不足20次，即使所有的请求都超时或其他原因失败，断路器都不会打开。
3. 错误百分比阀值：当请求总数在快照时间窗内超过了阀值，比如发生了30次调用，如果在这30次调用中，有15次发生了超时异常，也就是超过50%的错误百分比，在默认设定50%阀值情况下，这时候就会将断路器打开。