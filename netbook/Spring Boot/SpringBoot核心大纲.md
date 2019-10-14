# 核心技术大纲

## 一.核心特性

1. 组件自动装配: Web MVC、 Web Flux、JDBC等
2. 嵌入式Web容器: Tomcat、Jety以及 Undertow
3. 生产准备特性:指标、健康检查、外部化配置等

## 二.组件自动装配

- 激活:@ Enableauto Configuration
- 配置:/META-INF/ spring. factories
- 实现: XXXAUTO Confiquration

## 三.嵌入式Web容器

- Web Servlet: Tomcat、 Jetty和 Undertow
- Web Reactive: Netty Web Server

## 四.生产准备特性

- 指标:/ actuator/ metrics
- 健康检查:/ actuator/ health
- 外部化配置:/ actuator/ configprops

----------------

# Web应用

## 一.传统 Servlet应用

- Servlet组件: Servlet、 Filter、 Listener
- Servlet注册: Servlet注解、 Spring Bean、 Registrationbean
- 异步非阻塞:异步 Servlet、非阻塞 Servlet

## 1.组件

- Servlet
  - 实现
    - ` @WebServlet`
    - HttpServlet
    - 注册
  - URL映射
    - ` @WebServlet(urlPatterns="")`
  - 注册
    - ` @ServletComponentScan(basePackages='')`
- Filter
- Listener

## 二.Spring Web MVC

- Web MVC视图:模板引擎、内容协商、异常处理等
-  Web MVC REST:资源服务、资源跨域、服务发现等
- Web MVC核心:核心架构、处理流程、核心组件

### 1.Web MVC视图

- `VeiwResolver`

- `View`

#### a.模板引擎

- Thymeleaf
- Freemarker
- jsp

#### b.内容协商

- `ContentNegotiationConfigurer`

- `ContentNegotiationStrategy`
- ``ContentNegotiationViewResolver`

#### c.异常处理

- `@ExceptionHandler`
- `HanlerExceptionResolver`
  - `ExceptionHandlerEceptionResolver`

- `BasicErrorController`(Spring Boot)

 ### 2.Web MVC REST

#### a.资源服务

- `@RequestMapping`
  - `@GetMapping`

- `@ResponseBody`
- ` @RequestBody`

#### b.资源跨域

- `CrossOrigin`
- `WebMvcConfigurer#addCorsMappings`

#### c.服务发现

- HATEOS

### 3.Web MVC 核心

#### a. 核心架构

#### b.处理流程

#### c.核心组件

- `DispatCherServlet`
- `HandlerMapping`
- `HandlerAdapter`
- `ViewResolver`

## 三.Spring Web Flux应用

### 1.Reactor基础

#### a.Java Lambda

#### b.Mono

#### c.Flux

### 2.Web Flux 核心

#### a.Web MVC 注解兼容

- `@Controller`
- `@RequestMappring`
- `@ResponseBody`
- `@ResquestBody` 

#### b.函数式声明

- `RouterFunction`

#### c.异步非阻塞

- Servlet 3.1+
- Netty Reactor

### 3.使用场景

#### a.页面渲染

#### b.REST应用

#### c.性能测试

## 四.Web Server 应用

### 1.切换Web Server

#### a.切换其他Servlet 容器

- Tomcat->Jetty(Tomcat 依赖优先级最高)

#### b.替换Servlet容器

- WebFlux(依赖优先级低于传统模式)

### 2.自定义Servlet容器

- `WebServerFactoryCustomizer`

### 3.自定义Reactive Web Server

- `ReactiveWebServerFactoryCustomizer`

-----------------

# 数据相关

## 一.关系型数据

### 1.JDBC

#### a.依赖

#### b.数据源

- `javax.sql.DataSource`
- `JdbcTemplate`

#### c.自动装配

- `DataSourceAutoConfiguration`

### 2.JPA

#### a.依赖 

#### b.实体类的映射

- `javax.persistenc.OneToOne`
- `javax.persistenc.OneToMany`
- `javax.persistenc.ManyToOne`
- `javax.persistenc.ManyToMany`

#### c.实体操作

- `javax.persistenc.EntityManager`

#### d.自动装配

- `HiberNateJpaAutoConfiguration`

### 3.事务

#### a.依赖

#### b.Spring事务抽象

- `PlatformTransactionManager`

#### c.JDBC事务处理

- `DataSourceTransactionManager`

#### d.自动装配

- `TeansactionAuroConfiguration`

--------------------

# 功能扩展

## 一.SpringBoot应用

### 1.SpringApplication

#### a.失败分析

- `FailureAnalysisReporter`

#### b.应用特性

- `SpringApplication`

### c.Spring Boot配置

- 外部化配置
- `@Profile`
- 配置属性











