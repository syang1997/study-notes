# springfox(三)了解SpringPlugin

Spring Plugin提供一个标准的`Plugin<S>`接口供开发人员继承使用声明自己的插件机制,然后通过`@EnablePluginRegistries`注解依赖注入到Spring的容器中,Spring容器会为我们自动匹配到插件的所有实现子对象,最终我们在代码中使用时,通过依赖注入注解，注入`PluginRegistry<T extends Plugin<S>, S>`对象拿到插件实例进行操作。

`Plugin<S>`接口声明了一个接口实现,标注实现该插件是否支持，因为有可能存在多个接口实现的情况

## 一.示例

### 需求逻辑

话费充值示例：

1. 为指定用户充值话费
2. 如果是老用户则充值超过一百则返还10%

### 依赖

```gradle
dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
    compileOnly 'org.projectlombok:lombok:1.18.8'
    compile "org.springframework:spring-core:4.0.9.RELEASE"
    compile "org.springframework.plugin:spring-plugin-core:1.2.0.RELEASE"
    compile "org.slf4j:slf4j-api:1.7.21"
    compile "org.slf4j:jcl-over-slf4j:1.7.21"
    compile "ch.qos.logback:logback-classic:1.2.3"
    compile group: 'org.mapstruct', name: 'mapstruct-jdk8', version: '1.2.0.Final'
    compile group: 'org.mapstruct', name: 'mapstruct-processor', version: '1.2.0.Final'
}
```

### 基础实现

```java
@Data
public class MobileCustomer {

    //电话号码
    private String tel;

    //是否为老用户
    private Boolean old=false;

    public MobileCustomer(String tel) {
        this.tel = tel;
    }
}
```

```java
public interface MobileIncarementBusiness {

    //充值接口
    void increment(MobileCustomer mobileCustomer,Integer money);
}
```

```java
@Slf4j
public class MobileIncrementV1 implements MobileIncarementBusiness {
    @Override
    public void increment(MobileCustomer mobileCustomer, Integer money) {
        log.info("给{}充值电话费,充值金额:{}",mobileCustomer.getTel(),money);
        log.info("充值完成.");
        if (mobileCustomer.getOld()){
            log.info("老用户折扣");
            if (money>100){
                BigDecimal big=new BigDecimal(money).multiply(new BigDecimal(0.1));
                log.info("当前充值金额>100元,返冲{}元",big.intValue());
            }
        }
    }
}
```

```java
@Service
public class CustomerService {

    @Autowired
    MobileIncarementBusiness mobileIncrementV1;

    public void increments(MobileCustomer mobileCustomer,Integer money){
        mobileIncrementV1.increment(mobileCustomer,money);
    }
}
```

```java
@Configuration
public class MobileConfig {

    @Bean
    public MobileIncrementV1 mobileIncrementV1(){
        return new MobileIncrementV1();
    }
}
```

### 使用plugin

充值接口继承Plugin接口

```java
public interface MobileIncarementBusinessV2 extends Plugin<MobileCustomer> {

    //充值接口
    void increment(MobileCustomer mobileCustomer, Integer money);
}
```

v2接口实现业务接口也要同时实现Plugin的supports接口，作用为是否启用。

```java
@Slf4j
public class MobileIncrementV2 implements MobileIncarementBusinessV2 {
    @Override
    public void increment(MobileCustomer mobileCustomer, Integer money) {
        log.info("给{}充值电话费,充值金额:{}",mobileCustomer.getTel(),money);
        log.info("充值完成.");
    }

    @Override
    public boolean supports(MobileCustomer delimiter) {
        return true;
    }
}
```

我们使用Plugin的方式来进行充值返利

```java
@Slf4j
public class MobileIncrementDiscountv2 implements MobileIncarementBusinessV2 {
    @Override
    public void increment(MobileCustomer mobileCustomer, Integer money) {
        if (supports(mobileCustomer)){
            log.info("老用户折扣");
            if (money>100){
                if (money>100){
                    BigDecimal big=new BigDecimal(money).multiply(new BigDecimal(0.1));
                    log.info("当前充值金额>100元,返冲{}元",big.intValue());
                }
            }
        }
    }

    /**
     * 老用户则启用
     * @param delimiter
     * @return
     */
    @Override
    public boolean supports(MobileCustomer delimiter) {
        return delimiter.getOld();
    }
}
```

业务层自动注入PluginRegistry用于获取指定的所有的插件示例，从而进行业务处理。

```java
@Service
public class CustomerServiceV2 {
    @Autowired
    private PluginRegistry<MobileIncarementBusinessV2, MobileCustomer> mobileCustomerPluginRegistry;

    public void increments(MobileCustomer mobileCustomer,int money){
        //获取插件
        List<MobileIncarementBusinessV2> plugins=mobileCustomerPluginRegistry.getPlugins();
        for (MobileIncarementBusinessV2 incrementBusiness:plugins){
            //对人员进行充值
            incrementBusiness.increment(mobileCustomer,money);
        }
    }
}
```

```java
@Configuration
@EnablePluginRegistries({MobileIncarementBusinessV2.class})
public class MobileConfigV2 {

    @Bean
    public MobileIncrementV2 mobileIncrementV2(){
        return new MobileIncrementV2();
    }

    @Bean
    public MobileIncrementDiscountv2 mobileIncrementDiscountv2(){
        return new MobileIncrementDiscountv2();
    }
}
```

#### 结果

```java
public class Application {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context=
                new AnnotationConfigApplicationContext("com.demo.plugin");
        CustomerService customerService=context.getBean(CustomerService.class);
        CustomerServiceV2 customerServiceV2=context.getBean(CustomerServiceV2.class);
        MobileCustomer mobileCustomer=new MobileCustomer("13567662664");
        mobileCustomer.setOld(true);
        customerService.increments(mobileCustomer,120);
        customerServiceV2.increments(mobileCustomer,120);
    }
}
```

- 通过插件的方式,不需要更改原来已经稳定的业务代码,对系统稳定性来说尤为重要(系统稳定是基础)
- 与业务解耦,如果业务发生变化(在某个周期内)，或者有新用户的活动,我们只需要构建我们的业务代码,核心框架层无需更改
- 程序架构更清晰,分层设计更明显.