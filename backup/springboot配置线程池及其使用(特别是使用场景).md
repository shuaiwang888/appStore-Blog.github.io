## 前言
最近在做订单模块，用户购买产品之后，需要经过Kafak中间件发送后消费，下单成功之后需要给用户发送提醒短信。考虑发短信耗时的情况所以我想用异步的方法去执行，于是就在网上看见了Spring的@Async了。

但是遇到了许多问题，使得@Async无效，也一直没有找到很好的文章去详细的说明@Async的正确及错误的使用方法及需要注意的地方，这里简单整理了一下遇见的问题，Sring是以配置文件的形式来开启@Async，而SpringBoot则是以注解的方式开启。

我们可以使用springBoot默认的线程池，不过一般我们会自定义线程池（因为比较灵活），配置方式有：

- 使用 xml 文件配置的方式

- 使用Java代码结合@Configuration进行配置（推荐使用）

## 下面为将直接使用结合@Configuration进行配置：
```java
@Configuration
public class ThreadPoolConfig {

    @Bean(name = "taskExecutor")
    public ThreadPoolTaskExecutor adminMessageExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setThreadFactory(new ThreadFactoryBuilder().setNameFormat("admin-message-pool-%d").build());
        executor.setQueueCapacity(100);
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        return executor;
    }
}
```

## 创建两个异步方法的类，如下所示:

第一个类（这里模拟取消订单后发短信，有两个发送短信的方法）：

```java
@Service
public class TranTest2Service {
    Logger log = LoggerFactory.getLogger(TranTest2Service.class);
 
    // 发送提醒短信 1
        @PostConstruct // 加上该注解项目启动时就执行一次该方法
    @Async("taskExecutor")
    public void sendMessage1() throws InterruptedException {
        log.info("发送短信方法---- 1 执行开始");
        Thread.sleep(5000); // 模拟耗时
        log.info("发送短信方法---- 1 执行结束");
    }
 
    // 发送提醒短信 2
        @PostConstruct // 加上该注解项目启动时就执行一次该方法
    @Async("taskExecutor")
    public void sendMessage2() throws InterruptedException {
 
        log.info("发送短信方法---- 2 执行开始");
        Thread.sleep(2000); // 模拟耗时
        log.info("发送短信方法---- 2 执行结束");
    }
}
```

**代码中的 @Async("taskExecutor") 对应我们自定义线程池中的 @Bean("taskExecutor") ，表示使用我们自定义的线程池。**

第二个类。调用发短信的方法 （异步方法不能与被调用的异步方法在同一个类中，否则无效）：

```java
@Service
public class OrderTaskServic {
    @Autowired
    private TranTest2Service tranTest2Service;
 
    // 订单处理任务
    public void orderTask() throws InterruptedException {
 
        this.cancelOrder(); // 取消订单
        tranTest2Service.sendMessage1(); // 发短信的方法 1
        tranTest2Service.sendMessage2(); // 发短信的方法 2
 
    }
 
    // 取消订单
    public void cancelOrder() throws InterruptedException {
        System.out.println("取消订单的方法执行------开始");
        System.out.println("取消订单的方法执行------结束 ");
    }
}
```

**运行截图：**

![image.png](/upload/2022/01/image-6ef7c1582a1a4e92a679fe65f36d6872.png)

注意看，截图中的 [nio-8090-exec-1] 是Tomcat的线程名称

[Async-Service-1]、[Async-Service-2]表示线程1和线程2 ，是我们自定义的线程池里面的线程名称，我们在配置类里面定义的线程池前缀：

```java
private static final String threadNamePrefix = "Async-Service-"; // 线程池名前缀，说明我们自定义的线程池被使用了。
```

## 注意事项

**如下方式会使@Async失效:**

- 异步方法使用static修饰

- 异步类没有使用@Component注解（或其他注解）导致spring无法扫描到异步类

- 异步方法不能与被调用的异步方法在同一个类中

- 类中需要使用@Autowired或@Resource等注解自动注入，不能自己手动new对象

- 如果使用SpringBoot框架必须在启动类中增加@EnableAsync注解

## 总结
**IS-PARKING项目中使用场景：**
- 利用定时任务同步车位信息；
- 发送消息sendCardMessage();

**XZ—SHOP商城项目中的使用场景：**
- 超管或商家对于订单的导出操作；
