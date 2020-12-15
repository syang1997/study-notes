# Spring Boot Event

**事件与直接方法调用**

事件和直接方法调用都适合于不同的情况。对于方法调用，这就像断言一样，无论发送和接收模块的状态如何，他们都需要知道此事件的发生。

另一方面，对于事件，我们只是说发生了一个事件，并且通知了哪些模块不是我们关心的问题。当我们想将处理传递给另一个线程时，最好使用事件（例如：在完成某些任务时发送电子邮件）。同样，事件对于测试驱动的开发非常有用。



事件用于在松耦合的组件之间交换信息。由于发布者和订阅者之间没有直接耦合，因此我们可以修改订阅者而不影响发布者，反之亦然。让我们看看如何在Spring Boot应用程序中创建，发布和收听自定义事件。

## 1.创建一个ApplcationEvent

从Spring 4.2开始，我们还可以将对象直接发布为事件，而无需扩展ApplicationEvent：

## 2.发布一个 ApplicationEvent

我们使用ApplicationEventPublisher接口来发布事件：

```java
@Component
class Publisher {
  
  private final ApplicationEventPublisher publisher;
    
    Publisher(ApplicationEventPublisher publisher) {
      this.publisher = publisher;
    }

  void publishEvent(final String name) {
    // Publishing event created by extending ApplicationEvent
    publisher.publishEvent(new UserCreatedEvent(this, name));
    // Publishing an object as an event
    publisher.publishEvent(new UserRemovedEvent(name));
  }
}
```

当我们发布的对象不是ApplicationEvent时，Spring会自动用PayloadApplicationEvent包装它

## 3. 监听事件

有两种定义侦听器的方法。我们可以使用@EventListener注释或实现ApplicationListener接口。无论哪种情况，监听器类都必须由Spring管理。

从Spring 4.1开始，现在可以简单地注释托管bean的方法，@EventListener以自动注册ApplicationListener与该方法的签名匹配的方法：

```java
@Component
class UserRemovedListener {

  @EventListener
  ReturnedEvent handleUserRemovedEvent(UserRemovedEvent event) {
    // handle UserRemovedEvent ...
    return new ReturnedEvent();
  }

  @EventListener
  void handleReturnedEvent(ReturnedEvent event) {
        // handle ReturnedEvent ...
  }
  ...
}
```

启用注释驱动的配置时，不需要其他配置。我们的方法可以监听多个事件，或者如果我们想完全不使用任何参数来定义它，那么事件类型也可以在注释本身上指定。范例：@EventListener({ContextStartedEvent.class, ContextRefreshedEvent.class})。



对于带有注释@EventListener的方法的返回类型如定义为非void，Spring会将结果作为新事件发布给我们。在上面的示例中，ReturnedEvent第一种方法返回的结果将被发布，然后由第二种方法处理。

如果指定SpEL，Spring仅在某些情况下允许触发我们的侦听器condition：

```java
@Component
class UserRemovedListener {

  @EventListener(condition = "#event.name eq 'reflectoring'")
  void handleConditionalListener(UserRemovedEvent event) {
    // handle UserRemovedEvent
  }
}
```

仅当表达式的计算结果为true，或包含以下字符串之一时：“true”, “on”, “yes”, 或“1”.方法参数通过其名称公开。条件表达式还公开了一个引用了raw ApplicationEvent（#root.event）和实际方法参数的“根”变量(#root.args)

在以上示例中，UserRemovedEvent仅当#event.name的值为时'reflectoring'，才会触发侦听器。



侦听事件的另一种方法是实现ApplicationListener接口：

```java
@Component
class UserCreatedListener implements ApplicationListener<UserCreatedEvent> {

  @Override
  public void onApplicationEvent(UserCreatedEvent event) {
    // handle UserCreatedEvent
  }
}
```

## 4.异步事件监听器

默认情况下，spring事件是同步的，这意味着发布者线程将阻塞，直到所有侦听器都完成对事件的处理为止。

要使事件侦听器以异步模式运行，我们要做的就是@Async在该侦听器上使用注释：

```java
@Component
class AsyncListener {

  @Async
  @EventListener
  void handleAsyncEvent(String event) {
    // handle event
  }
}
```

为了使@Async注释生效，我们还必须注释一个@Configuration类，使用@EnableAsync注释SpringBootApplication类。

上面的代码示例还显示，我们可以将String用作事件。使用风险自负。最好使用特定于我们用例的数据类型，以免与其他事件冲突。

## 5.事务绑定事件

Spring允许我们将事件侦听器绑定到当前事务的某个阶段。当当前事务的结果对侦听器很重要时，这使事件可以更灵活地使用。

当我们使用注释我们的方法时@TransactionalEventListener，我们得到了一个扩展的事件监听器，该监听器知道事务：

```java
@Component
class UserRemovedListener {

  @TransactionalEventListener(phase=TransactionPhase.AFTER_COMPLETION)
  void handleAfterUserRemoved(UserRemovedEvent event) {
    // handle UserRemovedEvent
  }
}
```

UserRemovedListener 仅在当前事务完成时才调用。

我们可以将侦听器绑定到事务的以下阶段：



- AFTER_COMMIT：成功提交事务后，将处理该事件。如果事件侦听器仅在当前事务成功时才运行，则可以使用此方法。
- AFTER_COMPLETION：在事务提交或回滚时将处理该事件。例如，我们可以使用它在事务完成后执行清理。
- AFTER_ROLLBACK：交易回滚后，将处理该事件。
- BEFORE_COMMIT：

## 6.Spring Boot的应用程序事件

以上是Spring事件，Spring Boot提供了几个预定义ApplicationEvent的，这些预定义绑定到SpringApplication生命周期。

在ApplicationContext创建之前会触发一些事件，因此我们无法将这些事件注册为@Bean。我们可以通过手动添加侦听器来注册这些事件的侦听器：

```java
@SpringBootApplication
public class EventsDemoApplication {

  public static void main(String[] args) {
    SpringApplication springApplication = 
        new SpringApplication(EventsDemoApplication.class);
    springApplication.addListeners(new SpringBuiltInEventsListener());
    springApplication.run(args);
  }

}
```

通过将META-INF/spring.factories文件添加到我们的项目中，我们还可以注册侦听器，而不管如何创建应用的。并通过以下org.springframework.context.ApplicationListener键引用侦听器：

org.springframework.context.ApplicationListener= com.reflectoring.eventdemo.SpringBuiltInEventsListener

一旦确保正确注册了事件监听器，我们就可以监听所有Spring Boot的SpringApplicationEvents。让我们按照它们应用程序启动期间的执行顺序来看看：



**ApplicationStartingEvent**

ApplicationStartingEvent在运行开始时但在任何处理之前都会触发，除了侦听器和初始化程序的注册外。



**ApplicationEnvironmentPreparedEvent**

当Environment在上下文中是可用的，一个 被触发，由于此时Environment将准备就绪，因此我们可以在其他bean使用它之前对其进行检查和修改。



**ApplicationContextInitializedEvent**

ApplicationContext已准备就绪时，一个ApplicationContextInitializedEvent触发，ApplicationContextInitializers被称为尚未加载bean定义。在bean初始化到Spring容器之前，我们可以使用它执行任务。



**ApplicationPreparedEvent**

当ApllicationContext准备就绪时，一个ApplicationPreparedEvent时会触发，但不会刷新。

在准备好的Environment和bean定义将被加载。



**ContextRefreshedEvent**

当ApplicationContext刷新时，ContextRefreshedEvent会触发。

ContextRefreshedEvent是直接来自Spring，而不是Spring Boot，并不继承扩展SpringApplicationEvent。



**WebServerInitializedEvent**

如果我们使用的是Web服务器，WebServerInitializedEvent则在Web服务器准备就绪后会触发a。ServletWebServerInitializedEvent和ReactiveWebServerInitializedEvent分别是servlet和反应式变量。

WebServerInitializedEvent不是继承扩展SpringApplicationEvent。



**ApplicationStartedEvent**

上下文已被刷新之后，一个ApplicationStartedEvent触发，但在任何Spring boot应用程序和命令行运行都被调用前。



**ApplicationReadyEvent**

一个ApplicationReadyEvent触发时就表示该应用程序已准备好服务请求。

建议此时不要修改内部状态，因为所有初始化步骤都将完成。



**ApplicationFailedEvent**

一个ApplicationFailedEvent如果有异常，应用程序无法启动点火。在启动期间的任何时间都可能发生这种情况。我们可以使用它来执行一些任务，例如执行脚本或在启动失败时发出通知。

# 源码

## 1.ApplicationEventPublisher

![image-20200828133402838](imgs/image-20200828133402838.png)

```java
    @Nullable
    private Set<ApplicationListener<?>> earlyApplicationListeners;
    @Nullable
    private Set<ApplicationEvent> earlyApplicationEvents;    

protected void publishEvent(Object event, @Nullable ResolvableType eventType) {
        Assert.notNull(event, "Event must not be null");
        Object applicationEvent;
        //扩展了ApplicationEvent的
        if (event instanceof ApplicationEvent) {
            applicationEvent = (ApplicationEvent)event;
        } else {
        //未扩展的spring帮我们扩展
            applicationEvent = new PayloadApplicationEvent(this, event);
            if (eventType == null) {
                eventType = ((PayloadApplicationEvent)applicationEvent).getResolvableType();
            }
        }
		//加入到early事件集合中
        if (this.earlyApplicationEvents != null) {
            //这个事件集合会在refresh方法中被初始化后置空，同时也会并初始化earlylistener。
            this.earlyApplicationEvents.add(applicationEvent);
        } else {
            //添加到事件多播器中
            this.getApplicationEventMulticaster().multicastEvent((ApplicationEvent)applicationEvent, eventType);
        }
		//如果有父context则也进行发布，只到AbstractApplicationContext层
        if (this.parent != null) {
            if (this.parent instanceof AbstractApplicationContext) {
                ((AbstractApplicationContext)this.parent).publishEvent(event, eventType);
            } else {
                this.parent.publishEvent(event);
            }
        }

    }

```

```java
public class PayloadApplicationEvent<T> extends ApplicationEvent implements ResolvableTypeProvider {
    private final T payload;
	//帮我们包装了applicationEvent所以在4.2之后不需要扩展了，spring已经帮我们扩展了。
    public PayloadApplicationEvent(Object source, T payload) {
        super(source);
        Assert.notNull(payload, "Payload must not be null");
        this.payload = payload;
    }

    public ResolvableType getResolvableType() {
        return ResolvableType.forClassWithGenerics(this.getClass(), new ResolvableType[]{ResolvableType.forInstance(this.getPayload())});
    }

    public T getPayload() {
        return this.payload;
    }
}
```

 所以实际上event是被发布到了`ApplicationEventMulticaster`中。如果我们没有实现接口则spring会启动默认的`SimpleApplicationEventMulticaster`

所有的对listener的操作在`AbstractApplicationEventMulticaster`中实现。

![image-20200831101936625](imgs/image-20200707193527281.png)

```java
public class SimpleApplicationEventMulticaster extends AbstractApplicationEventMulticaster {
    public void multicastEvent(ApplicationEvent event, @Nullable ResolvableType eventType) {
        ResolvableType type = eventType != null ? eventType : this.resolveDefaultEventType(event);
        Executor executor = this.getTaskExecutor();
        Iterator var5 = this.getApplicationListeners(event, type).iterator();
		//根据event，和type来获取所有的listeners，taskExecutor不为空则是异步执行
        while(var5.hasNext()) {
            ApplicationListener<?> listener = (ApplicationListener)var5.next();
            if (executor != null) {
                executor.execute(() -> {
                    this.invokeListener(listener, event);
                });
            } else {
                this.invokeListener(listener, event);
            }
        }
    }
}
```

对listener的操作在abstracet中被定义。

**SmartApplicationListener **

SmartApplicationListener 接口是 ApplicationListener 的子接口，还继承了 Ordered 接口。SmartApplicationListener 定义了两个 support 方法用于判断事件类型、来源类型是否和当前监听者匹配，这样监听者可以筛选自己感兴趣的事件和来源。继承 Ordered 接口后，该监听者具备了排序的功能，可以按照 order 从小到大的顺序给监听者确定一个优先级，从而确保执行顺序。

**GenericApplicationListener**

GenericApplicationListener 接口是 ApplicationListener 的子接口，也继承了 Ordered 接口，同 SmartApplicationListener 一样具有事件筛选能力和排序能力。但筛选事件使用的是 ResolvableType 类型，而不是 ApplicationEvent 类型。