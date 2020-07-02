# Sentinel

## 一.简介

随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

Sentinel 具有以下特征:

- **丰富的应用场景**：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等。
- **完备的实时监控**：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
- **广泛的开源生态**：Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
- **完善的 SPI 扩展点**：Sentinel 提供简单易用、完善的 SPI 扩展接口。您可以通过实现扩展接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等。

## 二.环境搭建

下载sentinel-dashboard，并运行，默认为localhost：8080。

pom依赖

```xml
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
        <!--后续持久化-->
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>
```

yml

```yaml
spring:
  application:
    name: cloud-alibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
    sentinel:
      transport:
        dashboard: localhost:8080   #配置到dashboard的端口
        port: 8719        #默认端口8719，如果被占用则+1直到找到未被占用
```

## 三.使用

#### 1.流控规则

1. 资源名：唯一名称，默认请求路径
2. 针对来源：Sentinel可以针对调用者进行限流，填写微服务名，默认 default（不区分来源）
3. 阈值类型单机阈值
   * QPS（侮每秒钟的请求数量）：当调用该ap的QPS达到阈值的时候，进行限流
   * 线程数：当调用该ap的线程数达到阈值的时候，进行限流
4. 是否集群：不需要集
5. 群流控模式
   * 直接：api达到限流条件时，直接限流
   * 关联：当关联的资源达到阈值时，就跟流自己
   * 链路：只记录指定链路上的流量（指定资源从入口資源进来的流量，如果达到阈值，就进行限流）【ap级别的针对来源】
6. 流控效果
   * 快速失败：直接失败，抛异常
   * Warm Up：根据 cofaCtor（冷加载因子，默认3）的值，从阈值 codeFactor，经过预热时长，才达到设置的QPS阈值
   * 排队等待：匀速排队，让请求以匀速的速度通过，阈值类型必须设置为QPS，否则无效

### 2.降级

- 平均响应时间 (`DEGRADE_GRADE_RT`)：当 1s 内持续进入 N 个请求，对应时刻的平均响应时间（秒级）均超过阈值（`count`，以 ms 为单位），那么在接下的时间窗口（`DegradeRule` 中的 `timeWindow`，以 s 为单位）之内，对这个方法的调用都会自动地熔断（抛出 `DegradeException`）。注意 Sentinel 默认统计的 RT 上限是 4900 ms，**超出此阈值的都会算作 4900 ms**，若需要变更此上限可以通过启动配置项 `-Dcsp.sentinel.statistic.max.rt=xxx` 来配置。
- 异常比例 (`DEGRADE_GRADE_EXCEPTION_RATIO`)：当资源的每秒请求量 >= N（可配置），并且每秒异常总数占通过量的比值超过阈值（`DegradeRule` 中的 `count`）之后，资源进入降级状态，即在接下的时间窗口（`DegradeRule` 中的 `timeWindow`，以 s 为单位）之内，对这个方法的调用都会自动地返回。异常比率的阈值范围是 `[0.0, 1.0]`，代表 0% - 100%。
- 异常数 (`DEGRADE_GRADE_EXCEPTION_COUNT`)：当资源近 1 分钟的异常数目超过阈值之后会进行熔断。注意由于统计时间窗口是分钟级别的，若 `timeWindow` 小于 60s，则结束熔断状态后仍可能再进入熔断状态。

