# Spring Cloud Alibaba

## 一、注册中心-Nacos（AP）

### 1、[官方wiki](https://nacos.io/zh-cn/docs/what-is-nacos.html)

### 2、主要功能
![](https://img-blog.csdnimg.cn/1013e6f1566b4d569ce994e3dcaab86c.png)

服务的注册与发现是由 `Nacos discovery` 组件来实现的！ 其核心功能：
1. 服务的注册： Nacos-client发送REST请求向Nacos-server注册自己的服务，包括ip地址、端口、健康检查URL等元信息，收到后存在一个双层的内存map注册表中，当然你也可以存在MySQL关系型数据库中！
2. 服务的心跳：默认5s一次；
3. 服务同步：Nacos集群中的互相同步服务实例；
4. 服务发现：消费者在调用提供者时，也会发REST请求定时拉取注册服务清单，并且缓存在本地；
5. 服务健康检查：Nacos 提供对服务的实时的健康检查，阻止向不健康的主机或服务实例发送请求。Nacos 支持传输层 (PING 或 TCP)和应用层 (如 HTTP、MySQL、用户自定义）的健康检查。超过5s无心跳则改变状态，超过30s无心跳则进行剔除，当然了也有类似于Eurka的实例保护机制；

> Eurka的实例保护机制： Eureka Server 在运行期间会去统计心跳失败比例在 15 分钟之内是否低于 85%，如果低于 85%，Eureka Server 会将这些实例保护起来，让这些实例不会过期，但是在保护期内如果服务刚好这个服务提供者非正常下线了，此时服务消费者就会拿到一个无效的服务实例，此时会调用失败，对于这个问题需要服务消费者端要有一些容错机制，如重试，断路器等。

### 3、几种注册中心的对比：

![](https://img-blog.csdnimg.cn/4d7397233ebf45aeb37509e8d67708dd.png)

注意：
1. 在某种条件下（出现实例大面积下线），也不会直接将其实例剔除，也是在请求后访问，只是访问不可达而已，只是为了避免大面积服务下线后，对现目前健康实例造成大量并发访问，使得健康的实例也会宕机而出现全面崩溃；而那些访问不可达的实例，可以采取适当措施：重试、熔断降级等以防止雪崩！
2. Nacos其实和Fiegn一样，底层调用者都是RestTemplate的HTTP调用，且都是整合了Ribbon，使用的都是Ribbon的负载均衡策略（默认使用复合策略（地区+轮询），也有权重轮询、随机、Hash分配、最小活跃数等）；
3. 其中在生成RestTemolate实例上使用注解@LoadBaland，即可与Ribbon集成；

### 4、常用配置
![](https://img-blog.csdnimg.cn/6094af02ff4d47708ec3c0fd17070f97.png)

1. spring.application.name=stock-service 就是DataID的配置名称；
2. server-addr就是Nacos注册中心的服务地址。
3. nameSpace 当然了也可以配置它来对相同特征的服务进行分类的分组管理，如测试环境、预发环境、线上环境等；
4. 还有ephemeral、group、weight等等；

### 5、nacos的集群配置

![](https://img-blog.csdnimg.cn/5e70ebbfe8a84c99b077087028c6944f.png)

1. 下载集群Nacos；
2. 对每个Nacos的配置文件进行编辑：application.properties（包括server.port，确定数据源（集群一般使用外置数据源MySQL）；
3. 对cluster.conf进行配置ip：port；
4. 根据需求对startup.sh启动文件修改JVM参数；

**注意：以上配置其实并不能直接在java服务中配置具体的Nacos服务地址，需要在前面加一个服务端均衡器 Nginx，实现负载均衡；**

5. Nginx反向代理配置

![](https://img-blog.csdnimg.cn/2dbcc30f28c546a7afc9addaff182d4b.png)

如图：
- 在配置文件中，首先通过server配置Nginx对外提供的一个监听的端口，一旦有请求访问，就会通过规则匹配，反向代理访问；
- 如果存在多种规则，通过编写匹配规则location，来判断需要代理的哪个访问地址；
- 而上述需要反向代理的多个实例地址需在upsteam中配置，默认轮询其中的地址；
- 这样只需在配置文件中配置Nginx监听的地址，当请求访问该地址时便可以实现反向代理；

![](https://img-blog.csdnimg.cn/5e1fb39f7d8d4a21bd02eb6dd4edc762.png)

### 6、负载均衡Ribbon与nginx

![](https://img-blog.csdnimg.cn/649747bda7f54171a96be440a9f410d3.png)

1. Ribbon是Netflix实现的一套客户端负载均衡工具，Ribbon客户端组件提供了一系列的完善配置，如超时、重试等。通过Loadbalancer获取到服务提供的所有机器实例，Ribbon自动基于某种规则（轮询、随机等）去调用这些服务。当然Ribbon也可以实现我们自定义的负载均衡算法。

例如在Ribbon中，客户端会有一个服务地址的列表，在发送请求前通过负载均衡算法去选择一个服务实例，然后进行访问，即在客户端进行负载均衡算法的分配（客户端负载均衡）；

![](https://img-blog.csdnimg.cn/f733994925554f75aa3d7bd3003a7a0b.png)

2. 服务端的负载均衡：如Nginx，通过其进行负载均衡，先发送请求，然后通过负载均衡算法在多个服务器之间选择一个实例进行访问，即在服务器端再进行负载均衡算法分配；

![](https://img-blog.csdnimg.cn/135015ead05044ad8dc047000cdb0412.png)

3. 常见的负载均衡算法：随机、轮询、加权轮询（权重）、地址Hash、最小活跃数、重试策略等；
4. 其中Ribbon默认是使用复合判断server实例所在区域的性能和其可用性来选择服务器，过滤后采用线性轮询的方式；
5. 修改负载均衡策略的方式有两种：一是通过实现配置类，实现IRule接口，然后再启动类上进行参数配置；二是配置文件，yml的方式（推荐）；
6. 自定义均衡策略：

![](https://img-blog.csdnimg.cn/7544f85556f549f0b10152cf1a5d2629.png)

7. RestTemplate的发送请求流程：

- 如果是多个商品服务集群，首先就是必须要选择其中一个可用的服务器（客户端负载），如果是localhost:8082

- 选择完成后就在订单服务内替换对应的URL请求地址："http://localhost:8082/api/v1/product/find?id="productId

- 最后才是发送http请求： RestTemplate 执行它的 doExcute方法，其实内部还是使用的HttpUrlConnection

- 上述总结： 在发送请求之前需要对请求进行拦截处理：主要实现的方式就是添加拦截器，核心注解就是@LoadBalanced（添加负载均衡算法：就是在商品服务器列表中选择一个可用的）

8. 上述使用的Ribbon负载均衡，其中的远程调用都通过REST实现：RestTemplate.getForObject()，始终存在硬编码问题。所以在http远程调用中推荐使用Open Feign；

### 7、微服务调用Feign

Open Feign 其实是对Feign进行了增强，使其支持Spring MVC 注解，另外整合了Ribbon和Nacos，可以直接使用负载均衡！

步骤：
1. 添加依赖；
2. 添加API接口，注解@FeignClient，设置name、url等参数，再继承调用的方法，但不会实现具体逻辑；
3. 在启动类上使用注解@EnableFeignClients，会扫描当前包及其子包下贴了@FeignClient的接口并创建jdk默认动态代理对象；
4. 这样便可以直接在服务代码中直接调用；

> **其使用到动态代理，调用代理对象的方法invoker()，底层是通过FeignLoadBalancer来帮我们实现远程调用，本质还是使用了Robbin，我们在配置文件中配置Robbin的负载均衡策略依然是可以生效的，server-id查找使用@FeignClient中的name属性，也就是服务提供者produce-server，通过HttpURLConnection把我们发送远程调用请求。**

#### Fiegn的自定义配置：

1. 日志级别：用来检查接口调用是否失败、参数不正确等；
2. 超时时间：包括连接超时时间：connectTimeout和读取超时时间：readTimeout，那么我们为什么要设置超时时间呢？ **因为在多个服务之间的调用时，防止部分服务无法返回而影响整个系统！**
3. 超时重试次数机制：这就是在超时时间的基础上设置的措施；
4. 自定义拦截器实现认证逻辑；

## 二、Nacos配置中心

### 1、[官方wiki](https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-config)

### 2、主流配置中心的对比

配置中心主要的特点包括：可维护性、时效性、安全性；

1. SpringCloud Config，其大部分场景结合git进行使用，动态变更配置还需要依赖SpringCloud Bus消息总线来通知所有的客户端配置的变化；
2. Config不提供可视化界面；
3. Nacos Config使用HTTP长轮询，可在1s内感知并更新配置；

### 3、Nacos Config 配置中心的原理

更新配置后拉取配置的如何实现？ **每次配置更改后都会生成一个单独的MD5标志存储，客户端会间隔1s时间去对比自身缓存的MD5与配置中心的MD5是否一样，如果发现不同，则会直接拉取配置中心的最新配置。并将新的MD5缓存到本地！**

### 4、Nacos概念
1. 命名空间(Namespace)：命名空间可用于进行不同环境的配置隔离。一般一个环境划分到一个命名空间；

2. 配置分组(Group)：配置分组用于将不同的服务可以归类到同一分组。一般将一个项目的配置分到一组；

3. 配置集(Data ID)：在系统中，一个配置文件通常就是一个配置集。一般微服务的配置就是一个配置集，一般命名为微服务名加配置文件格式后缀；

![](https://img-blog.csdnimg.cn/b22c9bde1d394734ad5a6fe99582823a.png)

### 5、使用

1. 配置动态刷新

我们虽然实现了远程配置的存储，但中途对配置文件进行修改，我们的程序是无法读取到的，因此，我们需要开启配置的动态刷新功能。推荐使用注解的方式，当然也可以是直接在bootstrap.properties文件中进行配置：
```java
@RestController
@RefreshScope //只需要在需要动态读取配置的类上添加此注解就可以
public class NacosConfigController {

    @Value("${config.appName}")
    private String appName;

    @GetMapping("/nacos-config-test2")
    public String nacosConfingTest2() {
       return appName;
   }
}

```

2. 配置共享

当配置越来越多的时候，我们就发现有很多配置是重复的，这时候就考虑可不可以将公共配置文提取出来，然后实现共享呢？当然是可以的。接下来我们就来探讨如何实现这一功能。

同一个微服务的不同环境之间共享配置？ 如果想在同一个微服务的不同环境之间实现配置共享，其实很简单。

只需要提取一个以 spring.application.name命名的配置文件，然后将其所有环境的公共配置放在里面即可。

1、新建一个名为order.yaml配置存放商品微服务的公共配置

2、新建一个名为order-test.yaml配置存放测试环境的配置

3、新建一个名为order-dev.yaml配置存放开发环境的配置

4、添加测试方法

```java
@RestController
@RefreshScope
public class NacosConfigController {
    @Value("${config.env}")
    private String env;
    //同一微服务的不同环境下共享配置
    @GetMapping("/nacos-config-test3")
    public String nacosConfingTest3() {
      return env;
   }
}
```
接下来，修改bootstrap.yml中的配置，将active设置成test，再次访问，观察结果
```yml
spring:
profiles:
   active: test # 环境标识
```

### 6、xz-shop的配置实例

```
spring:
  profiles: local                                               # 此应该为命名空间，使用当前项目的不同环境（local、dev、preonline、production）来区分
  cloud:
    nacos:
      server-addr: is-nacos-dev.eff.internal:8848               # 服务器地址，Nacos Server监听的IP和端口（IP：端口）
      config:
        enabled: true                                           # 开启Nacos配置自动配置
        refresh-enabled: true                                   # 用于支持配置的动态更新
        context-path: nacos                                     # 上下文路径
        config-retry-time: 8000                                 # 重试超时时间（也可以配置获取配置超时时间：spring.cloud.nacos.config.timeout）
        enable-remote-sync-config: true
        fileExtension: yaml                                     # POS-1 确定自己的 dataID 后缀 yaml、yml，也可以是Properties格式
        namespace: 596410f8-babc-4c3d-92a5-61b1350ff92f         # POS-2 namespaceName/namespaceID 要区分，这里是 namespaceID
        group: DEFAULT_GROUP                                    # POS-3 这个不用改
        username: ${NACOS_USER:nacos}
        password: ${NACOS_PASSWORD:ykJBskxyHXuqSNhoTVd9x4EO}
        prefix: xz-shop
```


## 三、Sentinel

### 1、[官方wiki](https://sentinelguard.io/zh-cn/docs/introduction.html)

### 2、分布式系统可能遇到的问题

- 激增流量：峰值流量访问；
- 被其他服务拖垮：慢SQL、第三方调用无响应、线程堆积；
- 还有异常未处理、单点故障、负载不均、线程池满等；

> **总体来说：缺乏依赖隔离、缺乏高可用防护、缺乏容错机制（尤其是对流量的防护）------解决方案：Sentinel**

### 3、雪崩

![](https://img-blog.csdnimg.cn/c340668477a64c558e3d1035d7b12ef0.png)

**服务雪崩及其原因与解决（并不是缓存雪崩）：**

> **服务雪崩：** 多个服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其他的微服务，这就是所谓的“扇出”。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的“雪崩效应”

**服务雪崩原因：**
1. 激增流量导致系统CPU无法正常处理请求；
2. 激增流量打垮冷系统（数据库连接未创建、缓存未预热）；
3. 消息投递过快，导致消息处理积压；
4. 慢SQL查询耗尽连接池；
5. 第三方服务无响应；
6. 业务调用出现异常；

**服务雪崩如何解决：** 服务降级、服务熔断、服务隔离、限流模式、超时机制

1. 服务降级：在高并发情况下，防止用户一直等待（返回一个友好的提示，直接给客户端，不会去处理请求，调用fallBack本地方法），目的是为了用户体验。 秒杀---当前请求人数过多，请稍后重试；

2. 服务熔断：熔断机制目的为了保护服务，在高并发的情况下，如果请求达到一定极限(可以自己设置阔值)，如果流量超出了设置阈值，让后直接拒绝访问，保护当前服务。使用服务降级方式返回一个友好提示，服务熔断和服务降级一起使用。

> 熔断的设计主要参考了hystrix的做法，分为3个模块：熔断请求判断算法、熔断恢复机制、熔断报警机制
> - **熔断请求判断机制算法：** 使用无锁循环队列计数，每个熔断器默认维护10个bucket，每1秒一个bucket，每个blucket记录请求的成功、失败、超时、拒绝的状态，默认错误超过50%且10秒内超过20个请求进行中断拦截。
> - **熔断恢复机制：** 对于被熔断的请求，每隔5s允许部分请求通过，若请求都是健康的（RT<250ms）则对请求健康恢复。
> - **熔断报警机制：** 对于熔断的请求打日志，异常请求超过某些设定则报警。

3. 服务隔离：主要有 **线程池隔离** 和 **信号量隔离** 两种方式
- 线程池隔离: 每个服务接口，都有自己独立的线程池，每个线程池互不影响。 
- 信号量隔离: 使用一个原子计数器（或信号量）来记录当前有多少个线程在运行，当请求进来时先判断计数器的数值，若超过设置的最大线程个数则拒绝该请求，若不超过则通行，这时候计数器+1，请求返回成功后计数器-1。

4. 限流模式：上述都属于出错后的容错处理机制，而限流模式则可以称为预防模式。限流模式主要是提前对各个类型的请求设置最高的QPS阈值，若高于设置的阈值则对该请求直接返回，不再调用后续资源。这种模式不能解决服务依赖的问题，只能解决系统整体资源分配问题，因为没有被限流的请求依然有可能造成雪崩效应。

5. 超时机制：一般情况都是在Open Feign配置中设置超时时间（连接等待超时时间和运行读取超时时间）
- 连接等待超时：在任务入队列时设置任务入队列时间，并判断队头的任务入队列时间是否大于超时时间，超过则丢弃任务。
- 运行读取超时：直接可使用线程池提供的get方法

> 高可用防护组件------Sentinel：是面向分布式服务架构的流量控制组件，以流量为切入点，从限流、流量整形、熔断降级、系统负载保护、热点防护等多个维度来保障微服务的稳定性；

### 4、两种主流组件对比

强烈建议参考：[如何做技术选型？Sentinel 还是 Hystrix？
](https://github.com/doocs/advanced-java/blob/main/docs/high-availability/sentinel-vs-hystrix.md)

![](https://img-blog.csdnimg.cn/99b8eb02c3704e789370c82d3bd33be8.png)

- Sentinel主要是通过信号量进行隔离的，对于线程池的隔离会增加了线程切换的成本，还需要预先给各个资源做线程池大小的分配；
- 对于限流，Sentinel既可以基于QPS，也可以支持基于调用链路关系的限流；
- 而对于流量整形，Sentinel支持慢启动和匀速器两种模式，同时也支持系统负载保护；

### 5、基本使用

我们说的资源，可以是任何东西，服务，服务里的方法，甚至是一段代码。使用 Sentinel 来进行资源保护，主要分为几个步骤:

- 定义资源
- 定义规则
- 检验规则是否生效

先把可能需要保护的资源定义好，之后再配置规则。也可以理解为，只要有了资源，我们就可以在任何时候灵活地定义各种流量控制规则。在编码的时候，只需要考虑这个代码是否需要保护，如果需要保护，就将之定义为一个资源。

对于主流的框架，我们提供适配，只需要按照适配中的说明配置，Sentinel 就会默认定义提供的服务，方法等为资源。

![](https://img-blog.csdnimg.cn/bc0bef2a91074721ac9f4a696c6b7814.png)

1. 首先我们使用注解@SentinelResource的Value属性进行资源的定义，定义为USER-RESOUCE_NAME；
2. 其次是定义规则，以流控降级为例，配置均在Dashboard上，也可以直接使用代码的方式（不推荐），通过属性blockHandler定义一个方法，该方法为该资源被流控后的降级方法，注意需要捕捉BlockException异常；
3. 最后通过并发访问，检查规则是否生效；

> 在Dashboard上的配置是十分方便的，但是唯一的缺点就是：控制台上的配置一旦服务重启后就自动清除了，需要单独做持久化操作（Java代码集成后设置的规则是持久化的，但存在对业务的侵入）-----**结合Nacos配置，使得Sentinel去拉取配置，实现持久化；**

### 6、几种常见的规则方式

> **一、流量控制**

限流规则的几条因素：resource：资源名，即限流规则的作用对象；count: 限流阈值；grade: 限流阈值类型，QPS 或线程数；strategy: 根据调用关系选择策略。

![](https://img-blog.csdnimg.cn/8a5435c559f547399c8602c10c55e9f2.png)

#### 1. 基于 QPS/并发数的流量控制

流量控制主要有两种统计类型，一种是统计线程数，另外一种则是统计 QPS。

1.1 并发线程数流量控制

**线程数限流用于保护业务线程数不被耗尽。** 例如，当应用所依赖的下游应用由于某种原因导致服务不稳定、响应延迟增加，对于调用者来说，意味着吞吐量下降和更多的线程数占用，极端情况下甚至导致线程池耗尽。为应对高线程占用的情况，业内有使用隔离的方案，比如通过不同业务逻辑使用不同线程池来隔离业务自身之间的资源争抢（线程池隔离），或者使用信号量来控制同时请求的个数（信号量隔离）。这种隔离方案虽然能够控制线程数量，但无法控制请求排队时间。当请求过多时排队也是无益的，直接拒绝能够迅速降低系统压力。**Sentinel线程数限流不负责创建和管理线程池，而是简单统计当前请求上下文的线程个数，如果超出阈值，新的请求会被立即拒绝。**

1.2 QPS流量控制

当 QPS 超过某个阈值的时候，则采取措施进行流量控制。流量控制的手段包括下面 3 种，对应 FlowRule 中的controlBehavior 字段：

- 直接拒绝（RuleConstant.CONTROL_BEHAVIOR_DEFAULT）方式。该方式是默认的流量控制方式，**当QPS超过任意规则的阈值后，新的请求就会被立即拒绝，拒绝方式为抛出FlowException**。这种方式适用于对系统处理能力确切已知的情况下，比如通过压测确定了系统的准确水位时。具体的例子参见 FlowqpsDemo。

- 冷启动（RuleConstant.CONTROL_BEHAVIOR_WARM_UP）方式。该方式主要用于系统长期处于低水位的情况下，当流量突然增加时，直接把系统拉升到高水位可能瞬间把系统压垮。**通过"冷启动"，让通过的流量缓慢增加，在一定时间内逐渐增加到阈值上限，给冷系统一个预热的时间，避免冷系统被压垮的情况。** 通常冷启动的过程系统允许通过的 QPS 曲线如下图所示：

![](https://github.com/alibaba/Sentinel/wiki/image/warmup.gif)

- 匀速器（RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER）方式。这种方式严格控制了请求通过的间隔时间，也即是让请求以均匀的速度通过，**对应的是漏桶算法**。该方式的作用如下图所示：

![](https://github.com/alibaba/Sentinel/wiki/image/queue.gif)

这种方式主要用于处理间隔性突发的流量，例如消息队列。想象一下这样的场景，在某一秒有大量的请求到来，而接下来的几秒则处于空闲状态，我们希望系统能够在接下来的空闲期间逐渐处理这些请求，而不是在第一秒直接拒绝多余的请求。

### 2. 基于调用关系的流量控制

调用关系包括调用方、被调用方；方法又可能会调用其它方法，形成一个调用链路的层次关系。Sentinel 通过 NodeSelectorSlot 建立不同资源间的调用的关系，并且通过 ClusterNodeBuilderSlot 记录每个资源的实时统计信息。

2.1 根据调用方限流

2.2 根据调用链路入口限流：链路限流

2.3 具有关系的资源流量控制：关联流量控制

> **二、熔断降级**

除了流量控制以外，对调用链路中不稳定的资源进行熔断降级也是保障高可用的重要措施之一。一个服务常常会调用别的模块，可能是另外的一个远程服务、数据库，或者第三方 API 等。例如，支付的时候，可能需要远程调用银联提供的 API；查询某个商品的价格，可能需要进行数据库查询。然而，这个被依赖服务的稳定性是不能保证的。如果依赖的服务出现了不稳定的情况，请求的响应时间变长，那么调用服务的方法的响应时间也会变长，线程会产生堆积，最终可能耗尽业务自身的线程池，服务本身也变得不可用。

**因此我们需要对不稳定的弱依赖服务调用进行熔断降级，暂时切断不稳定调用，避免局部不稳定因素导致整体的雪崩。** 熔断降级作为保护自身的手段，通常在客户端（调用端）进行配置。

**Sentinel 提供以下几种熔断策略：**

- 慢调用比例 (SLOW_REQUEST_RATIO)：选择以慢调用比例作为阈值，需要设置允许的慢调用 RT（即最大的响应时间），请求的响应时间大于该值则统计为慢调用。当单位统计时长（statIntervalMs）内请求数目大于设置的最小请求数目，并且慢调用的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求响应时间小于设置的慢调用 RT 则结束熔断，若大于设置的慢调用 RT 则会再次被熔断。


- 异常比例 (ERROR_RATIO)：当单位统计时长（statIntervalMs）内请求数目大于设置的最小请求数目，并且异常的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。异常比率的阈值范围是 [0.0, 1.0]，代表 0% - 100%。


- 异常数 (ERROR_COUNT)：当单位统计时长内的异常数目超过阈值之后会自动进行熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。

> **三、系统自适应保护**

Sentinel 系统自适应保护从整体维度对应用入口流量进行控制，结合应用的 Load、总体平均 RT、入口 QPS 和线程数等几个维度的监控指标，让系统的入口流量和系统的负载达到一个平衡，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。

Sentinel 做系统自适应保护的目的：其一是保证系统不被拖垮，其二是在系统稳定的前提下，保持系统的吞吐量；

> **四、网关流量控制**

Sentinel 支持对 Spring Cloud Gateway、Zuul 等主流的 API Gateway 进行限流。

![](https://user-images.githubusercontent.com/9434884/58381714-266d7980-7ff3-11e9-8617-d0d7c325d703.png)

当通过 GatewayRuleManager 加载网关流控规则（GatewayFlowRule）时，无论是否针对请求属性进行限流，Sentinel 底层都会将网关流控规则转化为热点参数规则（ParamFlowRule），存储在 GatewayRuleManager 中，与正常的热点参数规则相隔离。转换时 Sentinel 会根据请求属性配置，为网关流控规则设置参数索引（idx），并同步到生成的热点参数规则中。

外部请求进入 API Gateway 时会经过 Sentinel 实现的 filter，其中会依次进行 路由/API 分组匹配、请求属性解析和参数组装。Sentinel 会根据配置的网关流控规则来解析请求属性，并依照参数索引顺序组装参数数组，最终传入 SphU.entry(res, args) 中。Sentinel API Gateway Adapter Common 模块向 Slot Chain 中添加了一个 GatewayFlowSlot，专门用来做网关规则的检查。GatewayFlowSlot 会从 GatewayRuleManager 中提取生成的热点参数规则，根据传入的参数依次进行规则检查。若某条规则不针对请求属性，则会在参数最后一个位置置入预设的常量，达到普通流控的效果。

> 五**、热点参数限流**

何为热点？热点即经常访问的数据。很多时候我们希望统计某个热点数据中访问频次最高的 Top K 数据，并对其访问进行限制。比如：

- 商品 ID 为参数，统计一段时间内最常购买的商品 ID 并进行限制
- 用户 ID 为参数，针对一段时间内频繁访问的用户 ID 进行限制


热点参数限流会统计传入参数中的热点参数，并根据配置的限流阈值与模式，对包含热点参数的资源调用进行限流。热点参数限流可以看做是一种特殊的流量控制，仅对包含热点参数的资源调用生效。

![](https://github.com/alibaba/Sentinel/wiki/image/sentinel-hot-param-overview-1.png)

**Sentinel 利用 LRU 策略统计最近最常访问的热点参数，结合令牌桶算法来进行参数级别的流控。**


## 四、分布式事务 Seata

事务的ACID：原子性、一致性、隔离性、持久性

事务分为本地事物：@Transaction 和分布式事务；

![](https://img-blog.csdnimg.cn/7cd5029958cb40d6a087252445535444.png)

在上述两种情况下，在使用常规本地事务方法均不能实现事务的统一性！ -------- 分布式事务解决方案：Seata

> Seata 是一款开源的分布式事务解决方案，致力于提供高性能和简单易用的分布式事务服务。Seata 将为用户提供了 AT、TCC、SAGA 和 XA 事务模式，为用户打造一站式的分布式解决方案。

基础理论可以阅读：[分布式事务六种解决方案](http://121.199.72.143:8090/archives/fen-bu-shi-shi-wu-liu-zhong-jie-jue-fang-an)

### 2PC

2PC模式主要有：Seata框架（首推AT模式）、消息队列（TCC）、SagaSaga、XA等，2PC主要分为**预处理**和**提交/回滚**两个阶段；

**1 预处理阶段：**
- 询问：协调者会向所有参与者发送事务请求，询问是否可以执行事务操作，等待各个参与者的响应；
- 执行：各参与者接收到协调者请求后，执行事务操作（如更新MySQL中的表记录），并将Undo，Redo休息记录到事务日志中；
- 响应：如果参与者正常执行第二步后，则向协调者返回Yes，否则返回No。当然也可能会宕机，从而无响应；

**2 提交/回滚阶段：**

- 将commit/rollback请求发给所有参与者；
- 参与者收到请求执行后，提交完成后释放事务所有占用的资源；
- 最后返回响应；

**上述存在的问题：**

- 会造成同步阻塞；
- 协调者会存在单点故障；
- 数据不一致：commit/rollback请求可能各种原因导致部分参与者未收到；
- 但仍是主流的事物模式；

> ### 一、AT模式：关注自己的业务SQL，做到对业务的无侵入，但 AT 模式是基于支持本地 ACID 事务 的 关系型数据库。

![](https://img-blog.csdnimg.cn/455e3f5f84994ca7bec3d253dade27f4.png)

**一阶段：**
- 在本阶段中，Seata会拦截”业务SQl“，首先解析SQL语句得到 SQL 的类型（UPDATE），表（product），条件（where name = 'TXC'）等相关的信息。再根据解析得到的条件信息，生成查询语句，定位数据。
- 在业务数据被更新前，将其查出来保存成“before image”：**前镜像**；
- 执行业务SQL更新业务数据，在业务数据更新后，再将其保存成“after image”：**后镜像**；
- 把前后镜像数据以及业务 SQL 相关的信息组成一条回滚日志记录，插入到 UNDO_LOG 表中。
- 最后生成行锁，以防止其他事务进来；
- 以上操作全部在一个数据库事务内完成，这样保证了一阶段操作的原子性；

![](https://img-blog.csdnimg.cn/0d88f5053fba4933ba65cc420114d26e.png)

**二阶段：**

![](https://img-blog.csdnimg.cn/9feb3373f6d04ad29b494aa145223856.png)

![](https://img-blog.csdnimg.cn/0a021013a76d41c595f2e9d3c2e689c9.png)

> ### 二、TCC模式：不依赖于底层数据资源的事务支持。一阶段的try，二阶段的commit/cancel都是自定义。

1. 代码侵入性比较强，并且得自己实现相关事务控制逻辑；
2. 整个过程没有锁，性能较高！

![](https://img-blog.csdnimg.cn/979a859833764fd0864ed98ba5d881f7.png)

> ### **三、消息队列实现可靠消息最终一致性方案：**

![](https://img-blog.csdnimg.cn/139ec2d45b3a4f26ae85c63e6b85c55e.png)

- try：首先由订单服务发送预备消息到MQ，但此时并不是将消息发送给库存服务；
- commit：尝试完成对数据库的操作，如果成功发送确认，否则执行cancel删除预备消息；
- cancel：删除在MQ中的预备消息；

如果MQ在收到预备消息后，迟迟收不到确认或删除消息的请求，会进行状态回查；

当订单服务确认消息后，MQ才会将事务消息投递给库存服务，完成扣减库存，其中有可能存在投递失败，可采取重试或者发送警告给开发者完成数据一致； 


## 五、网关Gateway组件

### 5.1 不用网关存在的问题：
1. 每个业务都需要鉴权、限流、权限校验、解决跨域问题等逻辑；
2. 代码需要维护许多的微服务域名；
3. 若后期对业务进行拆分成多个微服务，这是对外提供的服务也要拆分；

> Gateway 是Spring 生态系统之上构建的API网关服务，基于Spring 5,Spring Boot 2 和 Project Reactor等技术。Gateway 在提供一种简单而有效的方式来对API进行路由，以及提供一些强大的过滤器功能，例如：熔断，限流，重试等。

> SpringCloud Gateway 是基于WebFlux 框架实现的，而WebFlux框架底层则使用了高性能的Reactor 模式通讯框架Netty.

> Spring Cloud Gateway 的目标是提供统一的路由方式且基于Filter链的方式提供了网关的基本功能，例如：安全，监控/指标，和限流。

### 5.2 其衍生出来的功能主要包括：
- 动态路由：能够匹配任何请求属性
- 可以对路由指定Predicate(断言)和 Filter (过滤器)
- 全局性的流控
- 日志统计
- 防止SQl注入
- 防止web攻击
- 屏蔽工具扫描
- 黑白ip名单
- 证书/加解密处理
- 服务级别流控、服务降级与熔断、**路由与负载均衡**


### 5.3 三大核心概念
- **Route(路由)：** 路由是构建网关的基本模块，它由ID,目标URI,一系列的断言和过滤器组成，如果断言为true则匹配该路由
- **Predicate(断言)：** 参考的是Java8 的Java.util.funcation.Predicate, 开发人员可以匹配HTTP请求中的所有内容（例如请求头或请求参数），如果请求与断言相匹配则进行路由。
- **Filter(过滤器)：** 值的是Spring 框架中GatewatFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改

### 5.4 流程
客户端向Spring Cloud Gateway 发出请求。然后在Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到Gateway Web Handler。Handler 再通过制定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。

**微服务架构图**

![](https://img-blog.csdnimg.cn/00e5fa5a994943aba283ca3d1232a002.png)

![](https://img-blog.csdnimg.cn/cc58f3ac4f914122849373882455884b.png)

- 图中的Spring.application.name=api-gateway是网关在Nacos中的注册服务名，端口为8088；
- 在routes中配置路由规则：属性id代表路由目标地址的唯一标识，也是在注册中心注册的服务名；uri为路由的地址，也是目标地址；predicates为断言，当请求url满足下面各种规则就路由到上面的目标地址；filters过滤器，过滤掉转发路径中多余的标识，例如order-serv，这样的话真正的路由地址为 http://localhost:8020/order/add。

```yml
server:
  port: 8088

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      routes:   # 配置路由，是一个集合
        - id: payment-routh           # 路由的ID, 没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001  # 匹配后提供服务的路由地址
          predicates:
            - Path=/payment/**        # 断言，路径相匹配的进行路由转发
```

以上为示例配置单台gateway，一般情况下会像上面架构图中一样集成 Nacos！这样的话不能直接写死目标路由地址，所以可以用如下配置服务名代替：


![](https://img-blog.csdnimg.cn/9b1b5862f811429389102c3dc43e8ad6.png)

### 5.5 断言

当请求gateway时，使用断言对请求进行匹配，如果匹配成功就可以路由转发，失败就返回；其断言类型包括内置和自定义两种。

**内置：**
![](https://img-blog.csdnimg.cn/24e4c508f21a472ba689217ea03e6937.png)
![](https://img-blog.csdnimg.cn/3d617b16f0f6430393eabf7c0654383a.png)

- 其中根据Datetime类型、Cookie、Path等相对较多；
- 而一般在微服务框架中，首先客户端请求会先到Nginx进行服务器的负载均衡，将请求打到不同的网关，随后通过断言，网关会根据Nacos注册中心去获取路由目标，当然了目标路由地址也可能是集群，**所以会使用服务名进行客户端的负载均衡（Ribbon的轮询），当然了你也可以在网关层使用基于路由请求路径权重的内置断言工厂，实现可控的负载均衡。**

**自定义：**


![](https://img-blog.csdnimg.cn/e10b6497db2c43da933a299a7ab84e91.png)

自定义举例：实现一个name为xushu才能访问的断言！
![](https://img-blog.csdnimg.cn/d516cb9e9e6a483da10ee9e35abef05c.png)
![](https://img-blog.csdnimg.cn/0df315ba26fe4023af7c6f3713ace92f.png)

### 5.6 过滤器

**Filter：**

主要是在请求进来或者返回时做的操作！此为局部过滤器：主要是针对某个路由，需要在路由中配置！

**Global Filter**

此为全局过滤器，针对所有的路由请求！

![](https://img-blog.csdnimg.cn/f9f84ddcd05d4e7f8bee696061615d00.png)

### 5.7 请求日志处理

![](https://img-blog.csdnimg.cn/96176a2949f04c37ba60b559c9b0c771.png)

### 5.8 请求跨域

![](https://img-blog.csdnimg.cn/f28a640da8064b84bf69f707d44d3175.png)

## 六、链路追踪 Skywalking

服务监控需要满足的三要素分别如下：日志监控、指标监控、请求链路追踪；

场景：1. 用户请求调用微服务网关，请求到达后端服务。2. 此时用户有可能调用订单服务，订单服务有可能调用用户服务，用户服务有可能调用商品服务，商品服务有可能调用用户服务，用户服务有可能调用订单服务等。3. 在调用过程中存在很多请求跟踪问题，例如如果报错了，哪个环节出错了？这个问题如果不解决，很难快速定位bug。

### 6.1 链路追踪的作用：
- 定位上述场景的问题所在；
- 各微服务之间的依赖关系；
- 借口的性能分析；
- 业务流程的调用处理顺序；

总结来说就是：服务、服务实例、端点指标分析。服务拓扑图分析。慢查询检测；

![](https://img-blog.csdnimg.cn/d12bce71c3744cd9ab95e5ca82f77ee8.png)

### **6.2 SkyWalking 分为三个核心部分：**

**1. Agent（探针）：** Agent 运行在各个服务实例中，负责采集服务实例的 Trace 、Metrics 等数据，然后通过 gRPC 方式上报给 SkyWalking 后端。

**2. OAP：** SkyWalking 的后端服务，其主要责任有两个。

- 一个是负责接收 Agent 上报上来的 Trace、Metrics 等数据，交给 Analysis Core （涉及 SkyWalking OAP 中的多个模块）进行流式分析，最终将分析得到的结果写入持久化存储中。SkyWalking 可以使用 ElasticSearch、H2、MySQL 等作为其持久化存储，一般线上使用 ElasticSearch 集群作为其后端存储。

- 另一个是负责响应 SkyWalking UI 界面发送来的查询请求，将前面持久化的数据查询出来，组成正确的响应结果返回给 UI 界面进行展示。

**3. UI 界面：** SkyWalking 前后端进行分离，该 UI 界面负责将用户的查询操作封装为 GraphQL 请求提交给 OAP 后端触发后续的查询操作，待拿到查询结果之后会在前端负责展示。

> UI服务：默认8080，用于展示数据；
OAP-service：用于检测数据，暴露11800和12800两个端口；其中11800端口是收集探针采集的实例数据，12800端口是接受前端UI服务用于展示的请求；

### 6.3 开发部署

![](https://img-blog.csdnimg.cn/470ccbdb0a2341bb89276033fbbd21f9.png)

- 对于单个组件可以直接在IDEA中配置JVM参数；
- 而集群的话，只需要在每个微服务启动时添加Java agent参数即可；

上述是在整个服务进行链路追踪，当然也可以自定义的链路追踪，自定义适用于希望对项目中的特定业务方法进行监控，方便排查问题！

- 引入依赖：Skywaliking工具类；
- 在业务方法上加入注解@Teace：实现整个方法的监控；
- 也可以在业务方法加入@Tags/@Tag：实现以Key-Value的形式返回记录参数和信息；

### 6.4 监控数据持久化

目前默认是在h2内存数据库存储（缺点是一旦重启后数据丢失，内存较小服务器可能内存爆满），但也可以使用MySQL数据库持久化；

### 6.5 性能剖析

可以定位到当前业务方法下的某一句代码的响应时间，方便排查！

### 6.6 告警

![](https://img-blog.csdnimg.cn/f7aab1f4414a44e6a69a90f25294be34.png)

## 架构图

![](https://img-blog.csdnimg.cn/d7ba675f9dba42729b7e5a329e044383.png)