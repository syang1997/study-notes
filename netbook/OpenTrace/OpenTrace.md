# OpenTracing

## 一.简介

OpenTracing是一套开源的标准API，OpenTracing 是一个 Library，定义了一套通用的数据上报接口，要求各个分布式追踪系统都来实现这套接口。这样一来，应用程序只需要对接 OpenTracing，而无需关心后端采用的到底什么分布式追踪系统，因此开发者可以无缝切换分布式追踪系统，也使得在通用代码库增加对分布式追踪的支持成为可能。

https://github.com/opentracing-contrib/opentracing-specification-zh/blob/master/specification.md

opentracing中文文档 https://wu-sheng.gitbooks.io/opentracing-io/content/

https://blog.csdn.net/wanxiaoderen/article/details/107214981

https://blog.csdn.net/hanjiangxue1006/article/details/105626862



## 二.OpenTracing协议

###  1.概念和术语

```
一个tracer过程中，各span的关系


        [Span A]  ←←←(the root span)
            |
     +------+------+
     |             |
 [Span B]      [Span C] ←←←(Span C 是 Span A 的孩子节点, ChildOf)
     |             |
 [Span D]      +---+-------+
               |           |
           [Span E]    [Span F] >>> [Span G] >>> [Span H]
                                       ↑
                                       ↑
                                       ↑
                         (Span G 在 Span F 后被调用, FollowsFrom)
上述tracer与span的时间轴关系


––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–> time

 [Span A···················································]
   [Span B··············································]
      [Span D··········································]
    [Span C········································]
         [Span E·······]        [Span F··] [Span G··] [Span H··]
```

#### Traces

一个trace代表一个潜在的，分布式的，存在并行数据或并行执行轨迹（潜在的分布式、并行）的系统。一个trace可以认为是多个span的有向无环图（DAG）。

#### Spans

一个span代表系统中具有开始时间和执行时长的逻辑运行单元。span之间通过嵌套或者顺序排列建立逻辑因果关系。

####  Operation Names

每一个span都有一个操作名称，这个名称简单，并具有可读性高。（例如：一个RPC方法的名称，一个函数名，或者一个大型计算过程中的子任务或阶段）。span的操作名应该是一个抽象、通用的标识，能够明确的、具有统计意义的名称；更具体的子类型的描述，请使用[Tags](https://wu-sheng.gitbooks.io/opentracing-io/content/pages/spec.html#tags)

例如，假设一个获取账户信息的span会有如下可能的名称：

| 操作名            | 指导意见                                                     |
| :---------------- | :----------------------------------------------------------- |
| `get`             | 太抽象                                                       |
| `get_account/792` | 太明确                                                       |
| `get_account`     | 正确的操作名，关于`account_id=792`的信息应该使用[Tag](https://wu-sheng.gitbooks.io/opentracing-io/content/pages/spec.html#tags)操作 |

#### Inter-Span References

一个span可以和一个或者多个span间存在因果关系。OpenTracing定义了两种关系：`ChildOf` 和 `FollowsFrom`。**这两种引用类型代表了子节点和父节点间的直接因果关系**。未来，OpenTracing将支持非因果关系的span引用关系。（例如：多个span被批量处理，span在同一个队列中，等等）

**`ChildOf` 引用:** 一个span可能是一个父级span的孩子，即"ChildOf"关系。在"ChildOf"引用关系下，父级span某种程度上取决于子span。下面这些情况会构成"ChildOf"关系：

- 一个RPC调用的服务端的span，和RPC服务客户端的span构成ChildOf关系
- 一个sql insert操作的span，和ORM的save方法的span构成ChildOf关系
- 很多span可以并行工作（或者分布式工作）都可能是一个父级的span的子项，他会合并所有子span的执行结果，并在指定期限内返回

下面都是合理的表述一个"ChildOf"关系的父子节点关系的时序图。

```
    [-Parent Span---------]
         [-Child Span----]

    [-Parent Span--------------]
         [-Child Span A----]
          [-Child Span B----]
        [-Child Span C----]
         [-Child Span D---------------]
         [-Child Span E----]
```

**`FollowsFrom` 引用:** 一些父级节点不以任何方式依然他们子节点的执行结果，这种情况下，我们说这些子span和父span之间是"FollowsFrom"的因果关系。"FollowsFrom"关系可以被分为很多不同的子类型，未来版本的OpenTracing中将正式的区分这些类型

下面都是合理的表述一个"FollowFrom"关系的父子节点关系的时序图。

```
    [-Parent Span-]  [-Child Span-]


    [-Parent Span--]
     [-Child Span-]


    [-Parent Span-]
                [-Child Span-]
```

#### Logs

每个span可以进行多次**Logs**操作，每一次**Logs**操作，都需要一个带时间戳的时间名称，以及可选的任意大小的存储结构。

标准中定义了一些日志（logging）操作的一些常见用例和相关的log事件的键值，可参考[Data Conventions Guidelines 数据约定指南](https://wu-sheng.gitbooks.io/opentracing-io/content/pages/api/data-conventions.html)。

#### Tags

每个span可以有多个键值对（key:value）形式的**Tags**，**Tags**是没有时间戳的，支持简单的对span进行注解和补充。

和使用**Logs**的场景一样，对于应用程序特定场景已知的键值对**Tags**，tracer可以对他们特别关注一下。更多信息，可参考[Data Conventions Guidelines 数据约定指南](https://wu-sheng.gitbooks.io/opentracing-io/content/pages/api/data-conventions.html)。

#### SpanContext

每个span必须提供方法访问**SpanContext**。SpanContext代表跨越进程边界，传递到下级span的状态。(例如，包含`<trace_id, span_id, sampled>`元组)，并用于封装**Baggage** (关于Baggage的解释，请参考下文)。SpanContext在跨越进程边界，和在追踪图中创建边界的时候会使用。(ChildOf关系或者其他关系，参考[Span间关系](https://wu-sheng.gitbooks.io/opentracing-io/content/pages/spec.html#references) )。

#### Baggage

**Baggage**是存储在SpanContext中的一个键值对(SpanContext)集合。它会在一条追踪链路上的所有span内*全局*传输，包含这些span对应的SpanContexts。在这种情况下，"Baggage"会随着trace一同传播，他因此得名（Baggage可理解为随着trace运行过程传送的行李）。鉴于全栈OpenTracing集成的需要，Baggage通过透明化的传输任意应用程序的数据，实现强大的功能。例如：可以在最终用户的手机端添加一个Baggage元素，并通过分布式追踪系统传递到存储层，然后再通过反向构建调用栈，定位过程中消耗很大的SQL查询语句。

Baggage拥有强大功能，也会有很大的*消耗*。由于Baggage的全局传输，如果包含的数量量太大，或者元素太多，它将降低系统的吞吐量或增加RPC的延迟。

#### Baggage vs. Span Tags

- Baggage在全局范围内，（伴随业务系统的调用）跨进程传输数据。Span的tag不会进行传输，因为他们不会被子级的span继承。
- span的tag可以用来记录业务相关的数据，并存储于追踪系统中。实现OpenTracing时，可以选择是否存储Baggage中的非业务数据，OpenTracing标准不强制要求实现此特性。

#### Inject and Extract

SpanContexts可以通过**Injected**操作向**Carrier**增加，或者通过**Extracted**从**Carrier**中获取，跨进程通讯数据（例如：HTTP头）。通过这种方式，SpanContexts可以跨越进程边界，并提供足够的信息来建立跨进程的span间关系（因此可以实现跨进程连续追踪）。

#### 平台无关的API语义

OpenTracing支持了很多不同的平台，当然，每个平台的API试图保持各平台和语言的习惯和管理，尽量做到入乡随俗。也就是说，每个平台的API，都需要根据上述的核心tracing概念来建模实现。在这一章中，我们试图描述这些概念和语义，尽量减少语言和平台的影响。

#### The `Span` Interface

`Span`接口必须实现以下的功能：

- **Get the `Span`'s [`SpanContext`](https://wu-sheng.gitbooks.io/opentracing-io/content/pages/spec.html#SpanContext)**， 通过span获取SpanContext （即使`span`已经结束，或者即将结束）
- **Finish**，完成已经开始的`Span`。处理获取`SpanContext`之外，Finish必须是span实例的最后一个被调用的方法。**(py: `finish`, go: `Finish`)**。 一些的语言实现方法会在`Span`结束之前记录相关信息，因为Finish方法可能不会被调用，因为主线程处理失败或者其他程序错误。在这种情况下，实现应该明确的记录`Span`，保证数据的持久化。
- **Set a key:value tag on the `Span`.**，为`Span`设置tag。tag的key必须是`string`类型，value必须是`string`，`boolean`或数字类型。tag其他类型的value是没有定义的。如果多个value对应同一个key（例如被设置多次），实现方式是没有被定义的。**(py: `set_tag`, go: `SetTag`)**
- **Add a new log event**，为`Span`增加一个log事件。事件名称是`string`类型，参数值可以是任何类型，任何大小。tracer的实现者不一定保存所有的参数值（设置可以所有参数值都不保存）。其中的时间戳参数，可以设置当前时间之前的时间戳。**(py: `log`, go: `Log`)**
- **Set a Baggage item**, 设置一个string:string类型的键值对。注意，新设置的Baggage元素，只保证传递到未来的子级的`Span`。参考下图所示。**(py: `set_baggage_item`, go: `SetBaggageItem`)**
- **Get a Baggage item**， 通过key获取Baggage中的元素。**(py: `get_baggage_item`, go: `BaggageItem`)**

```
        [Span A]
            |
     +------+------+
     |             |
 [Span B]      [Span C] ←←← (1) BAGGAGE ITEM "X" IS SET ON SPAN C,BUT AFTER SPAN E ALREADY STARTED.
     |             |            为SPAN C设置BAGGAGE元素，值为X，时间点为SPAN E已经开始运行
 [Span D]      +---+-----+
               |         |
           [Span E]  [Span F] >>> [Span G] >>> [Span H]
                                                 ↑
                                                 ↑
                                                 ↑
             (2) BAGGAGE ITEM "X" IS AVAILABLE FOR RETRIEVAL BY SPAN H (A CHILD OF SPAN C), AS WELL AS SPANS F AND G.
                 SPAN C元素的F\G\H子级span可以读取到BAGGAGE的元素X
```

#### The `SpanContext` Interface

`SpanContext`接口必须实现以下功能。用户可以通过`Span`实例或者`Tracer`的Extract能力获取`SpanContext`接口实例。

- **Iterate over all Baggage items** 是一个只读特性。**(py: `baggage`, go: `ForeachBaggageItem`)**
- 虽然以前`SpanContext`是`Tracer`接口的一部分，但是`SpanContext`对于[Inject and Extract](https://wu-sheng.gitbooks.io/opentracing-io/content/pages/spec.html#inject-extract)是必不可少的。

#### The `Tracer` Interface

`Tracer`接口必须实现以下功能：

- **Start a new `Span`**, 创建一个新的Span。调用者可以指定一个或多个[`SpanContext` 关系](https://wu-sheng.gitbooks.io/opentracing-io/content/pages/spec.html#references)(例如 `FollowsFrom` 或 `ChildOf`关系)，显示声明一个开始的时间戳（除"now"之外），并设置处理化的`Span`的tags数据集。**(py: `start_span`, go: `StartSpan`)**
- **Inject a `SpanContext`**，将`SpanContext`注入到`SpanContext`对象中，用于进行跨进程的传输。"carrier"类型需要反射或者明确指定的方式来确定。查看[end-to-end propagation example 端到端传递示例](https://wu-sheng.gitbooks.io/opentracing-io/content/propagation#propagation-example)获取更多消息。
- **Extract a `SpanContext`** ，通过"carrier"跨进程获取`SpanContext`信息。Extract会检查`carrier`，尝试获取先前通过`Inject`放入的数据，并重建`SpanContext`实例。除非有错误发生，Extract返回一个包含`SpanContext`实例，此实例可以用来创建一个新的子级Span。（注意：一些OpenTracing实现方式，认为`Span`在RPC的两端应该具有相同的ID，而另一些考虑客户端是父级span，服务端是子级span）。"carrier"类型需要反射或者明确指定的方式来确定。查看[end-to-end propagation example 端到端传递示例](https://wu-sheng.gitbooks.io/opentracing-io/content/propagation#propagation-example)获取更多消息。

#### Global and No-op Tracers

每一个平台的OpenTracing API库 (例如 [opentracing-go](https://github.com/opentracing/opentracing-go), [opentracing-java](https://github.com/opentracing/opentracing-java)，等等；不包含OpenTracing `Tracer`接口的实现）必须提供一个no-op Tracer（不具有任何操作的tracer）作为接口的一部分。No-op Tracer的实现必须不会出错，并且不会有任何副作用，包括baggage的传递时，也不会出现任何问题。同样，Tracer的实现也必须提供no-op Span实现；通过这种方法，监控代码不依赖于Tracer关于Span的返回值，针对no-op实现，不需要修改任何源代码。No-op Tracer的Inject方法永远返回成功，Extract返回的效果，和"carrier"中没有找到`SpanContext`时返回的结果一样。

每一个平台的OpenTracing API库 *可能* 支持配置(Go: `InitGlobalTracer()`, py: `opentracing.tracer = myTracer`)和获取单例的全局Tracer实例(Go: `GlobalTracer()`, py: `opentracing.tracer`)。如果支持全局的Tracer，默认返回的必须是no-op Tracer。

## 三.opentracing-java-api

代码结构

![image-20200730143443809](E:\笔记\自己笔记\study-notes\netbook\OpenTrace\image-20200730143443809.png)

| 名称               | 描述                                                         | 依赖                                                     |
| ------------------ | ------------------------------------------------------------ | -------------------------------------------------------- |
| `opentracing-api`  | 是一个纯粹的API没有任何依赖                                  | -                                                        |
| `opentracing-noop` | 实现了API,但是是空实现什么也不干                             | `opentracing-api`                                        |
| `opentracing-util` | 包含了一个`GlobalTracer`和基于`Thread_local`的简单实现`ScopeManager` | `opentracing-api`; `opentracing.noop`                    |
| `opentracing-mock` | mock测试，包含一个简单的`MockTracer`，将数据存褚进内存       | `opentracing-api`; `opentracing.noop`;`opentracing-util` |

### 1.核心模块

![image-20200730145933854](E:\笔记\自己笔记\study-notes\netbook\OpenTrace\image-20200730145933854.png)

#### 1).Tracer

一个全局的调用链生成器，包含一个SpanBuilder。

![image-20200730150319891](E:\笔记\自己笔记\study-notes\netbook\OpenTrace\image-20200730150319891.png)

#### 2).ScopeManager

这个组件的作用是能够通过它获取当前线程中启用的Span信息，并且可以启用一些处于未启用状态的span。在一些场景中，我们在一个线程中可能同时建立多个span，但是同一时间统一线程只会有一个span在启用，其他的span可能处在下列的状态中：

1. 等待子span完成
2. 等待某种阻塞方法
3. 创建并未开始

于一个线程Thread来说，一个时刻只有一个`span`是激活状态的(active)。其他`spans`,可能是这个`span`的上游，或者等待资源（I/O或者childspan的完成）等，这些`spans`的特点是，开始了(start)但是没有结束(end)。
这个时候获取当前的`span`是很不方便的，所以就有`scopeManager()`来管理每个span的`scope`。获取当前的span可以使用

#### 3).Span

span是树形的，所以要么是root要么来源于另外一个span

一个新的span会以激活的span作为parent。如果想无视激活的span具体制定parent可以使用Tracer中SpanBuilder的ignoreActiveSpan()。

#### 4).SpanContext

表示一个span对应的上下文，span和spanContext基本上是一一对应的关系，上下文存储的是一些需要跨越边界的一些信息，例如：

1. spanId 当前这个span的id
2. traceId 这个span所属的traceId(也就是这次调用链的唯一id)
3. baggage 其他的能过跨越多个调用单元的信息
   这个SpanContext可以通过某些媒介和方式传递给调用链的下游来做一些处理（例如子Span的id生成、信息的继承打印日志等等）

#### 5).References

代表了span的关系。

- child_of 父子关系
- follows_from 兄弟关系，先后关系

#### 6).Scope

代表了一个激活和被激活状态的span 。
往往一个span从开始到finsh完成，期间要等chid_span先完成。child_span运行时，当前的span就处于一个没有激活的状态。
所以`Scope`就可以理解代表了附加状态的一个`span`，也被用来获取`span`。也负责一个span的关闭

#### 7).propagation

![image-20200730151450814](E:\笔记\自己笔记\study-notes\netbook\OpenTrace\image-20200730151450814.png)

负责 放入和解析的类。这里仅仅定义了接口，没有定义实现。

##### Carrier

存放消息的载体。

**Map结构的Carrier**
		`TextMap`是Map结构的`Carrier`

`TextMapInject` 定义了 put接口

`TextMapExtract` 定义了一个获取`Map`迭代器的方法。

`TextMapInjectAdapter` 接受map，并写入数据

`TextMapInjectAdapter` 接受map，获取迭代器

**二进制结构的Carrier**

首先`ByteBuffer`是java的nio的类，内置一个byte[]数组，length就是`Carrier`的大小。

+`injectionBuffer`是返回一个`ByteBuffer`对象，用于用于SpanContext的注入。当调用`Tracer.inject()`会调用到这个方法。

- `extractionBuffer`返回SpanContext的`Carrier。`Tracer.extract()`会调到。

##### Format

`TextMap`定义了，解析和放置的行为，比如解析RPC协议和Http协议，就是不同实现类，真正的定义的行为。
`Format`更像是一个工厂，里面保存了各种`TextMap`的实现。

![image-20200730151951428](E:\笔记\自己笔记\study-notes\netbook\OpenTrace\image-20200730151951428.png)

# Jeager

https://rocdu.gitbook.io/jaeger-doc-zh/architecture