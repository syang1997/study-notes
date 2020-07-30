# Spring Boot actuator

## 一.配置启动

### 1.maven 

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

### 2.gradle

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
}
```

## 二.端点

执行器端点允许您监视应用程序并与之交互。Spring Boot包括许多内置端点，并允许您添加自己的端点。例如，运行状况端点提供基本的应用程序运行状况信息。

每个端点都可以通过HTTP或JMX启用或禁用并公开(使其可远程访问)。端点在启用和公开时被认为是可用的。内置端点只有在可用时才会自动配置。大多数应用程序选择通过HTTP公开，其中端点的ID连同/致执行器前缀被映射到一个URL。例如，默认情况下，运行状况端点映射到/执行器/运行状况。

以下是与技术无关的端点:

| ID                 | Description                                                  |
| :----------------- | :----------------------------------------------------------- |
| `auditevents`      | 公开当前应用程序的审计事件信息。需要AuditEventRepository bean。 |
| `beans`            | 显示应用程序中所有Spring bean的完整列表。                    |
| `caches`           | 公开可用的缓存。                                             |
| `conditions`       | 显示在配置和自动配置类上评估的条件，以及它们匹配或不匹配的原因。 |
| `configprops`      | 显示所有@ConfigurationProperties的排序列表。                 |
| `env`              | 从Spring的ConfigurableEnvironment中公开属性。                |
| `flyway`           | 显示已应用的所有Flyway数据库迁移。需要一个或多个`Flyway` beans。 |
| `health`           | 显示应用程序运行状况信息。                                   |
| `httptrace`        | 显示HTTP跟踪信息(默认情况下，最后100个HTTP请求-响应交换)。需要HttpTraceRepository bean。 |
| `info`             | 显示任意的应用程序信息。                                     |
| `integrationgraph` | 显示Spring集成图。需要依赖于spring-integration-core。        |
| `loggers`          | 显示和修改应用程序中日志记录器的配置。                       |
| `liquibase`        | 显示已应用的任何Liquibase数据库迁移。需要一个或多个`Liquibase` beans。 |
| `metrics`          | 显示当前应用程序的“度量”信息。                               |
| `mappings`         | 显示所有@RequestMapping路径的排序列表。                      |
| `scheduledtasks`   | 显示应用程序中的计划任务。                                   |
| `sessions`         | 允许从支持会话的Spring会话存储中检索和删除用户会话。需要使用Spring Session的基于servlet的web应用程序。 |
| `shutdown`         | 让应用程序优雅地关闭。默认情况下禁用。                       |
| `threaddump`       | 执行线程转储。                                               |

如果你的应用程序是一个web应用程序(Spring MVC, Spring WebFlux，或Jersey)，你可以使用以下附加端点:

| ID           | Description                                                  |
| :----------- | :----------------------------------------------------------- |
| `heapdump`   | 返回一个hprof堆转储文件。                                    |
| `jolokia`    | 通过HTTP公开JMX bean(当Jolokia在类路径上时，WebFlux不能使用)。需要依赖joloka -core。 |
| `logfile`    | 返回日志文件的内容(如果log .file.name或log .file)。路径属性已经设置)。支持使用HTTP范围头来检索日志文件的部分内容。 |
| `prometheus` | 以Prometheus服务器可以获取的格式公开指标。需要依赖 `micrometer-registry-prometheus`。 |

### 1.启动端点

默认情况下，除了关机之外，所有端点都是启用的。

```properties
management.endpoint.shutdown.enabled=true
```

### 2.展示端点

要更改公开的端点，请使用以下特定于技术的include和exclude属性:

| Property                                    | Default        |
| :------------------------------------------ | :------------- |
| `management.endpoints.jmx.exposure.exclude` |                |
| `management.endpoints.jmx.exposure.include` | `*`            |
| `management.endpoints.web.exposure.exclude` |                |
| `management.endpoints.web.exposure.include` | `info, health` |

include属性列出公开的端点的id。exclude属性列出不应该公开的端点的id。排除属性优先于包括属性。可以使用端点id列表配置include和exclude属性。

例如，要停止通过JMX公开所有端点，而只公开运行状况和信息端点，请使用以下属性:

```pr
management.endpoints.jmx.exposure.include=health,info
```

*在YAML中有特殊的含义，所以如果您想包含(或排除)所有端点，请务必添加引号，如下例所示:

### 3.保护HTTP端点

应该像保护其他敏感URL一样保护HTTP端点。如果存在Spring安全性，则在默认情况下使用Spring安全性的内容协商策略保护端点。例如，如果您希望为HTTP端点配置自定义安全性，只允许具有特定角色的用户访问它们，Spring Boot提供了一些方便的RequestMatcher对象，可以与Spring security结合使用。

一个典型的Spring安全配置看起来像下面的例子:

```java
@Configuration(proxyBeanMethods = false)
public class ActuatorSecurity extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.requestMatcher(EndpointRequest.toAnyEndpoint()).authorizeRequests((requests) ->
                requests.anyRequest().hasRole("ENDPOINT_ADMIN"));
        http.httpBasic();
    }

}
```

前面的示例使用EndpointRequest.toAnyEndpoint()将请求匹配到任何端点，然后确保所有端点都具有ENDPOINT_ADMIN角色。EndpointRequest上还有其他几个匹配器方法。有关详细信息，请参阅API文档(HTML或PDF)。

如果在防火墙后部署应用程序，您可能希望不需要身份验证就可以访问所有执行器端点。您可以通过更改management.endpoints.web.exposure来做到这一点。

此外，如果存在Spring安全性，则需要添加自定义安全配置，允许未经身份验证访问以下示例所示的端点:

```java
@Configuration(proxyBeanMethods = false)
public class ActuatorSecurity extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.requestMatcher(EndpointRequest.toAnyEndpoint()).authorizeRequests((requests) ->
            requests.anyRequest().permitAll());
    }

}
```

### 4.配置端点

端点自动缓存不带任何参数的读取操作的响应。若要配置端点缓存响应的时间，请使用其缓存。生存时间属性。下面的示例将bean端点缓存的生存时间设置为10秒:

```properties
management.endpoint.beans.cache.time-to-live=10s
```

### 5.跨域

默认关闭,配置后开启

```properties
management.endpoints.web.cors.allowed-origins=https://example.com
management.endpoints.web.cors.allowed-methods=GET,POST
```

### 6.实现自定义端点

如果您添加了一个用@Endpoint注释的@Bean，那么任何用@ReadOperation、@WriteOperation或@DeleteOperation注释的方法都将在JMX上自动公开，在web应用程序中也可以通过HTTP公开。可以使用Jersey、Spring MVC或Spring WebFlux通过HTTP公开端点。如果Jersey和Spring MVC都可用，则将使用Spring MVC。

使用@JmxEndpoint或@WebEndpoint编写特定于技术的端点。这些端点受到各自技术的限制。例如，@WebEndpoint只通过HTTP公开，而不通过JMX公开。

| Operation          | HTTP method |
| :----------------- | :---------- |
| `@ReadOperation`   | `GET`       |
| `@WriteOperation`  | `POST`      |
| `@DeleteOperation` | `DELETE`    |

### 7.返回值

端点操作的默认响应状态取决于操作类型(读、写或删除)以及操作返回的内容(如果有的话)。

@ReadOperation返回一个值，响应状态为200 (OK)。如果它不返回值，响应状态将为404 (not Found)。

如果@WriteOperation或@DeleteOperation返回一个值，则响应状态为200 (OK)。如果不返回值，则响应状态为204(无内容)。

如果在没有必需参数或参数不能转换为必需类型的情况下调用操作，操作方法将不会被调用，响应状态将为400(坏请求)。

### 8.servlet & controller

@ControllerEndpoint @ServletEndpoint 会导致高耦合和不可移植不推荐使用

### 9.健康信息

management.endpoint.health。显示详细和management.endpoint.health。

| Name              | Description                                                  |
| :---------------- | :----------------------------------------------------------- |
| `never`           | 细节从未显示。                                               |
| `when-authorized` | 详细信息只显示给授权用户。可以使用management.endpoint.health.roles配置授权角色。 |
| `always`          | 详细信息将显示给所有用户。                                   |

#### 1).自动配置的健康检测

在配置了一些特定的数据库后，springboot会自动配置上其健康检测

| Name                                                         | Description                                               |
| :----------------------------------------------------------- | :-------------------------------------------------------- |
| [`CassandraHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/cassandra/CassandraHealthIndicator.java) | Checks that a Cassandra database is up.                   |
| [`CouchbaseHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/couchbase/CouchbaseHealthIndicator.java) | Checks that a Couchbase cluster is up.                    |
| [`DataSourceHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/jdbc/DataSourceHealthIndicator.java) | Checks that a connection to `DataSource` can be obtained. |
| [`DiskSpaceHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/system/DiskSpaceHealthIndicator.java) | Checks for low disk space.                                |
| [`ElasticSearchRestHealthContributorAutoConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/elasticsearch/ElasticSearchRestHealthContributorAutoConfiguration.java) | Checks that an Elasticsearch cluster is up.               |
| [`HazelcastHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/hazelcast/HazelcastHealthIndicator.java) | Checks that a Hazelcast server is up.                     |
| [`InfluxDbHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/influx/InfluxDbHealthIndicator.java) | Checks that an InfluxDB server is up.                     |
| [`JmsHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/jms/JmsHealthIndicator.java) | Checks that a JMS broker is up.                           |
| [`LdapHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/ldap/LdapHealthIndicator.java) | Checks that an LDAP server is up.                         |
| [`LivenessStateHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/availability/LivenessStateHealthIndicator.java) | Exposes the "Liveness" application availability state.    |
| [`MailHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/mail/MailHealthIndicator.java) | Checks that a mail server is up.                          |
| [`MongoHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/mongo/MongoHealthIndicator.java) | Checks that a Mongo database is up.                       |
| [`Neo4jHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/neo4j/Neo4jHealthIndicator.java) | Checks that a Neo4j database is up.                       |
| [`PingHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/PingHealthIndicator.java) | Always responds with `UP`.                                |
| [`RabbitHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/amqp/RabbitHealthIndicator.java) | Checks that a Rabbit server is up.                        |
| [`ReadinessStateHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/availability/ReadinessStateHealthIndicator.java) | Exposes the "Readiness" application availability state.   |
| [`RedisHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/redis/RedisHealthIndicator.java) | Checks that a Redis server is up.                         |
| [`SolrHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/solr/SolrHealthIndicator.java) | Checks that a Solr server is up.                          |

#### 2).自定义健康检测

```java
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class MyHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        int errorCode = check(); // perform some specific health check
        if (errorCode != 0) {
            return Health.down().withDetail("Error Code", errorCode).build();
        }
        return Health.up().build();
    }

}
```

如果需要其他的状态可以在`management.endpoint.health.status.order`中定义

```properties
management.endpoint.health.status.order=fatal,down,out-of-service,unknown,up
```

任何没有映射的状态都是对应的200，如果自定义的状态可以在配置中映射其返回码

```properties
management.endpoint.health.status.http-mapping.down=503
management.endpoint.health.status.http-mapping.fatal=503
management.endpoint.health.status.http-mapping.out-of-service=503
```

内置的状态映射如下

| Status         | Mapping                                      |
| :------------- | :------------------------------------------- |
| DOWN           | SERVICE_UNAVAILABLE (503)                    |
| OUT_OF_SERVICE | SERVICE_UNAVAILABLE (503)                    |
| UP             | No mapping by default, so http status is 200 |
| UNKNOWN        | No mapping by default, so http status is 200 |

#### 3).响应试健康监控

```java
@Component
public class MyReactiveHealthIndicator implements ReactiveHealthIndicator {

    @Override
    public Mono<Health> health() {
        return doHealthCheck() //perform some specific health check that returns a Mono<Health>
            .onErrorResume(ex -> Mono.just(new Health.Builder().down(ex).build()));
    }

}
```

| Name                                                         | Description                             |
| :----------------------------------------------------------- | :-------------------------------------- |
| [`CassandraReactiveHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/cassandra/CassandraReactiveHealthIndicator.java) | Checks that a Cassandra database is up. |
| [`CouchbaseReactiveHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/couchbase/CouchbaseReactiveHealthIndicator.java) | Checks that a Couchbase cluster is up.  |
| [`MongoReactiveHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/mongo/MongoReactiveHealthIndicator.java) | Checks that a Mongo database is up.     |
| [`RedisReactiveHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.3.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/redis/RedisReactiveHealthIndicator.java) | Checks that a Redis server is up.       |

## 三.通过http来进行监控和管理

### 1.自定义管理端点路径

```properties
management.endpoints.web.base-path=/
management.endpoints.web.path-mapping.health=healthcheck
```

### 2.自定义配置服务的端口

```properties
management.server.port=8081
```

### 3.禁用Http端口

```properties
management.server.port=-1

management.endpoints.web.exposure.exclude=*
```

## 四.日志

访问http://localhost:8080/actuator/loggers可获取到日志

post一个实体即可设置日志等级

```json
{
    "configuredLevel": "DEBUG"
}
```

日志等级一共有：

- `TRACE`
- `DEBUG`
- `INFO`
- `WARN`
- `ERROR`
- `FATAL`
- `OFF`
- `null`

## 五.指标

actuator为度量器提供依赖关系管理和自动配置，Micrometer是一个应用程序度量facade，支持多种监控系统，包括: