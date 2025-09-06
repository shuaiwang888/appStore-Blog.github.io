> 本文基于 Spring Cloud Netflix 。Spring Cloud Alibaba 也是非常不错的选择哦！

首先我给大家看一张图，如果大家对这张图有些地方不太理解的话，我希望你们看完我这篇文章会恍然大悟。

![springcloud总体架构.jpg](/upload/2021/10/spring-cloud%E6%80%BB%E4%BD%93%E6%9E%B6%E6%9E%84-a04bf973ff95492d9a6d2861a60bd5e8.jpg)

## 一、什么是Spring cloud

**Spring cloud是其Apache公司开发的原生微服务框架，目前主流实现有Spring cloud-Netflix和Spring cloud-Alibaba两种，其中Alibaba框架中，Nacos：服务注册与配置中心；Sentinel：服务监控降级；Seata：分布式事务；RocketMQ：分布式消息系统；Dubbo：RPC框架；Scheduler：分布式任务调度。**

> 构建分布式系统不需要复杂和容易出错。Spring Cloud 为最常见的分布式系统模式提供了一种简单且易于接受的编程模型，帮助开发人员构建有弹性的、可靠的、协调的应用程序。Spring Cloud 构建于 Spring Boot 之上，使得开发者很容易入手并快速应用于生产中。

官方果然官方，介绍都这么有板有眼的。

我所理解的 `Spring Cloud` 就是微服务系统架构的一站式解决方案，在平时我们构建微服务的过程中需要做如 **服务发现注册** 、**配置中心** 、**消息总线** 、**负载均衡** 、**断路器** 、**数据监控** 等操作，而 Spring Cloud 为我们提供了一套简易的编程模型，使我们能在 Spring Boot 的基础上轻松地实现微服务项目的构建。

### SpringBoot和SpringCloud有啥关系?

SpringBoot专注于快速方便的开发单个个体微服务。
 
SpringCloud是关注全局的微服务协调整理治理框架，它将SpringBoot开发的一个个单体微服务整合并管理起来，为各个微服务之间提供，配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等集成服务；
 
SpringBoot可以离开SpringCloud独立使用开发项目，但是SpringCloud离不开SpringBoot，属于依赖的关系。
 
SpringBoot专注于快速、方便的开发单个微服务个体，SpringCloud关注全局的服务治理框架。

### 什么是微服务？
简单来说，微服务架构风格是一种将**一个单一应用程序**开发为**一组小型服务**的方法，**每个服务运行在自己的进程中（都可以单独部署），服务间通信采用轻量级通信机制(通常用HTTP资源API)**。这些服务围绕业务能力构建并且可通过全自动部署机制独立部署。这些服务共用一个最小型的集中式的管理，服务可用不同的语言开发，使用不同的数据存储技术。

### 单体应用：

![01单体应用.jpg](/upload/2021/10/01%E5%8D%95%E4%BD%93%E5%BA%94%E7%94%A8-a0f9bca4f1da4e06b87970029d89bb73.jpg)

### 分布式系统：

![02分布式系统.jpg](/upload/2021/10/02%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F-554d0494a3c44467bd24f7bafc91d59c.jpg)

### 集群：
集群就是为了避免单点故障，多台服务器环境和提供的功能是一模一样的；

![03集群.jpg](/upload/2021/10/03%E9%9B%86%E7%BE%A4-91e6ed2ae3554eedb749b04c5f072a0e.jpg)

### 微服务：

![04微服务.jpg](/upload/2021/10/04%E5%BE%AE%E6%9C%8D%E5%8A%A1-3b3200f806ac4b8f882bfbd2eeb365b1.jpg)

### 分布式+微服务+集群

![05分布式微服务集群.jpg](/upload/2021/10/05%E5%88%86%E5%B8%83%E5%BC%8F+%E5%BE%AE%E6%9C%8D%E5%8A%A1+%E9%9B%86%E7%BE%A4-7982169cc1fa43d0801d154af9c2f67a.jpg)


### 微服务的好处？
1. 单个服务挂了，不至于所有的模块都不能用
2. 资源的合理分配，针对每个服务进行资源调整
3. 可以模块化开发，对于测试，开发都会比整个系统来说要简单

### 微服务面临的问题？
1. 由于引入了网络，网络存在不确定性
2. 对于不同服务之间的数据一致性（分布式事务）
3. 对于系统问题的日志跟踪比较复杂（日志跟踪方案ELK）
4. 增加了整个系统的复杂度

### 主流的微服务框架
1. SpringCloud
2. SpringCloud Alibaba
3. 阿里巴巴Dubbo/HSF
4. 京东JSF
5. 新浪微博Motan
6. 当当网Dubbox

**Dubbo与SpringCloud对比：**


![Dubbo和SpringCloud对比 3.png](/upload/2021/10/Dubbo%E5%92%8CSpringCloud%E5%AF%B9%E6%AF%94%203-f86d77c8bd154cb8a180359e525bcb1a.png)

### Spring Cloud 的版本

当然这个只是个题外话。

`Spring Cloud` 的版本号并不是我们通常见的数字版本号，而是一些很奇怪的单词。这些单词均为英国伦敦地铁站的站名。同时根据字母表的顺序来对应版本时间顺序，比如：最早 的 `Release` 版本 `Angel`，第二个 `Release` 版本 `Brixton`（英国地名），然后是 `Camden`、 `Dalston`、`Edgware`、`Finchley`、`Greenwich`、`Hoxton`。

## 二、Spring Cloud 的服务注册与发现框架——Eureka

> `Eureka`是基于`REST`（代表性状态转移）的服务，主要在 `AWS` 云中用于定位服务，以实现负载均衡和中间层服务器的故障转移。我们称此服务为`Eureka`服务器。Eureka还带有一个基于 `Java` 的客户端组件 `Eureka Client`，它使与服务的交互变得更加容易。客户端还具有一个内置的负载平衡器，可以执行基本的循环负载平衡。在 `Netflix`，更复杂的负载均衡器将 `Eureka` 包装起来，以基于流量，资源使用，错误条件等多种因素提供加权负载均衡，以提供出色的弹性。

总的来说，`Eureka` 就是一个服务发现框架。何为服务，何又为发现呢？

举一个生活中的例子，就比如我们平时租房子找中介的事情。

在没有中介的时候我们需要一个一个去寻找是否有房屋要出租的房东，这显然会非常的费力，一你找凭一个人的能力是找不到很多房源供你选择，再者你也懒得这么找下去(找了这么久，没有合适的只能将就)。**这里的我们就相当于微服务中的 `Consumer` ，而那些房东就相当于微服务中的 `Provider` 。消费者 `Consumer` 需要调用提供者 `Provider` 提供的一些服务，就像我们现在需要租他们的房子一样。**

但是如果只是租客和房东之间进行寻找的话，他们的效率是很低的，房东找不到租客赚不到钱，租客找不到房东住不了房。所以，后来房东肯定就想到了广播自己的房源信息(比如在街边贴贴小广告)，这样对于房东来说已经完成他的任务(将房源公布出去)，但是有两个问题就出现了。第一、其他不是租客的都能收到这种租房消息，这在现实世界没什么，但是在计算机的世界中就会出现 **资源消耗** 的问题了。第二、租客这样还是很难找到你，试想一下我需要租房，我还需要东一个西一个地去找街边小广告，麻不麻烦？

那怎么办呢？我们当然不会那么傻乎乎的，第一时间就是去找 **中介** 呀，它为我们提供了统一房源的地方，我们消费者只需要跑到它那里去找就行了。而对于房东来说，他们也只需要把房源在中介那里发布就行了。

那么现在，我们的模式就是这样的了。

![24382ce6bbd44932ac38b1accade12d1newimage2ff8affc6f1d49dea8c3801e7bad2b11.png](/upload/2021/10/24382ce6bbd44932ac38b1accade12d1-new-image2ff8affc-6f1d-49de-a8c3-801e7bad2b11-990102854a33411094b84d994ab4c923.png)

但是，这个时候还会出现一些问题。

1. 房东注册之后如果不想卖房子了怎么办？我们是不是需要让房东 **定期续约** ？如果房东不进行续约是不是要将他们从中介那里的注册列表中 **移除** 。
2. 租客是不是也要进行 **注册** 呢？不然合同乙方怎么来呢？
3. 中介可不可以做 **连锁店** 呢？如果这一个店因为某些不可抗力因素而无法使用，那么我们是否可以换一个连锁店呢？

针对上面的问题我们来重新构建一下上面的模式图

![租房中介模式图.jpg](/upload/2021/10/%E7%A7%9F%E6%88%BF-%E4%B8%AD%E4%BB%8B%E6%A8%A1%E5%BC%8F%E5%9B%BE-e838bccba2524056a25c9090f7012942.jpg)

好了，举完这个:chestnut:我们就可以来看关于 `Eureka` 的一些基础概念了，你会发现这东西理解起来怎么这么简单。:punch::punch::punch:

**服务发现**：其实就是一个“中介”，整个过程中有三个角色：**服务提供者(出租房子的)、服务消费者(租客)、服务中介(房屋中介)**。

**服务提供者**： 就是提供一些自己能够执行的一些服务给外界。

**服务消费者**： 就是需要使用一些服务的“用户”。

**服务中介**： 其实就是服务提供者和服务消费者之间的“桥梁”，服务提供者可以把自己注册到服务中介那里，而服务消费者如需要消费一些服务(使用一些功能)就可以在服务中介中寻找注册在服务中介的服务提供者。

### Eureka客户端与服务器之间的通信流程

**服务注册 Register**：

Eureka客户端将关于运行实例的信息注册到Eureka服务器。注册发生在第一次心跳

官方解释：当 `Eureka` 客户端向 `Eureka Server` 注册时，它提供自身的**元数据**，比如IP地址、端口，运行状况指示符URL，主页等。

结合中介理解：房东 (提供者 `Eureka Client Provider`)在中介 (服务器 `Eureka Server`) 那里登记房屋的信息，比如面积，价格，地段等等(元数据 `metaData`)。

**服务续约 Renew**：

官方解释：Eureka客户端需要更新最新注册信息（续借），通过每30秒发送一次心跳。更新通知是为了告诉Eureka服务器实例仍然存活。如果服务器在90秒内没有看到更新，它会将实例从注册表中删除。建议不要更改更新间隔，因为服务器使用该信息来确定客户机与服务器之间的通信是否存在广泛传播的问题！

结合中介理解：房东 (提供者 `Eureka Client Provider`) 定期告诉中介 (服务器 `Eureka Server`) 我的房子还租(续约) ，中介 (服务器`Eureka Server`) 收到之后继续保留房屋的信息。

**获取注册列表信息 Fetch Registries**： 

官方解释：`Eureka` 客户端从服务器获取注册表信息，并将其缓存在本地。客户端会使用该信息查找其他服务，从而进行远程调用。该注册列表信息定期（每30秒钟）更新一次。每次返回注册列表信息可能与 `Eureka` 客户端的缓存信息不同, `Eureka` 客户端自动处理。如果由于某种原因导致注册列表信息不能及时匹配，`Eureka` 客户端则会重新获取整个注册表信息。 `Eureka` 服务器缓存注册列表信息，整个注册表以及每个应用程序的信息进行了压缩，压缩内容和没有压缩的内容完全相同。`Eureka` 客户端和 `Eureka` 服务器可以使用JSON / XML格式进行通讯。在默认的情况下 `Eureka` 客户端使用压缩 `JSON` 格式来获取注册列表的信息。

结合中介理解：租客(消费者 `Eureka Client Consumer`) 去中介 (服务器 `Eureka Server`) 那里获取所有的房屋信息列表 (客户端列表 `Eureka Client List`) ，而且租客为了获取最新的信息会定期向中介 (服务器 `Eureka Server`) 那里获取并更新本地列表。

**服务下线 Cancel**：

官方解释：Eureka客户端在程序关闭时向Eureka服务器发送取消请求。 发送请求后，该客户端实例信息将从服务器的实例注册表中删除。该下线请求不会自动完成，它需要调用以下内容：`DiscoveryManager.getInstance().shutdownComponent();`

结合中介理解：房东 (提供者 `Eureka Client Provider`) 告诉中介  (服务器 `Eureka Server`) 我的房子不租了，中介之后就将注册的房屋信息从列表中剔除。

**服务剔除 Eviction**：

官方解释：在默认的情况下，**当Eureka客户端连续90秒(3个续约周期)没有向Eureka服务器发送服务续约，即心跳，Eureka服务器会将该服务实例从服务注册列表删除**，即服务剔除。

结合中介理解：房东(提供者 `Eureka Client Provider`) 会定期联系 中介  (服务器 `Eureka Server`) 告诉他我的房子还租(续约)，如果中介  (服务器 `Eureka Server`) 长时间没收到提供者的信息，那么中介会将他的房屋信息给下架(服务剔除)。

下面就是 `Netflix` 官方给出的 `Eureka` 架构图，你会发现和我们前面画的中介图别无二致。

![5d723c49eca1468ab7b89af06743023cnewimageb8aa3d41fad44b38add9c304930ab285.png](/upload/2021/10/5d723c49eca1468ab7b89af06743023c-new-imageb8aa3d41-fad4-4b38-add9-c304930ab285-2cb746545a24460581888f1c7a17a577.png)

当然，可以充当服务发现的组件有很多：`Zookeeper` ，`Consul` ， `Eureka` 等。

更多关于 `Eureka` 的知识(自我保护，初始注册策略等等)可以自己去官网查看，或者查看我的另一篇文章 [深入理解 Eureka](<https://juejin.im/post/5dd497e3f265da0ba7718018>)。

## Eureka实例

### 1. 为什么需要注册中心？

远程服务调用在没有注册中心前存在什么问题(画图讲解)：订单服务是无法直接获商品服务的url，就算在订单服务的配置中心写死了商品服务的url，也同样存在问题：如果后续对商品服务进行动态扩容呢！ 所以需要注册中心，在商品服务启动时，就去注册中心注册自己的ip和端口。

![QQ图片20211008103517.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211008103517-53c4c911bfc54db7b0bac611e6cb74a8.png)

### 2. 注册中心的作用？

微服务应用和机器越来越多，调用方需要知道接口的网络地址，如果靠配置文件的方式去控制网络地址，对于动态新增机器，url地址维护带来很大问题，注册中心提供服务注册与发现功能，对服务的url地址进行统一管理。

**对于服务提供者Provider的作用：** 启动的时候向注册中心上报自己的网络信息；

**对于服务消费者Consumer的作用：** 启动的时候向注册中心上报自己的网络信息，拉取provider的相关网络信息；

### 3. 主流的注册中心？
zookeeper、Eureka、consul、etcd、nacos等等

### 4. 案例项目-微服务基础模块设计

我们在讲解SpringCloud的课程中，使用简单易懂的模块给同学讲解里面相关的组件.

**商品服务：**
1. 查询商品列表
2. 查询商品详情

**订单服务：**
1. 创建订单

**案例功能：** 前台调用订单服务，订单服务远程调用商品服务获取商品详情信息，基于该商品信息创建订单。

### 5. 案例项目-注册中心eureka-server搭建

**步骤：**
- 使用Spring Initializr创建SpringBoot项目，选择Cloud Discover->Eureka Server

- 启动类上贴上@EnableEurekaServer注解

- 修改application.properties为application.yml文件，添加相关配置信息. https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.0.RELEASE/reference/html/#spring-cloud-eureka-server（下面有写）

- 运行测试，打开浏览器输入http://localhost:8761

**application.yml：**
```yml
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false  # 是否往注册中心注册信息（否，本身自己就是注册中心）
    fetchRegistry: false   # 是否往注册中心拉取信息
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

**依赖pom.xml：**
```xml
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.maoge.demo02</groupId>
    <artifactId>eureka-server-02</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>eureka-server-02</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.SR6</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

### 6. 案例项目-商品服务接口product-api搭建（放商品服务的公共资源)

**步骤：**

- 创建骨架类型为quickstart项目,删除多余的依赖和src/test目录的文件（不需要启动）

- 复制pom.xml

- 创建domain的product类

**pom.xml文件：** 略

**Product类：**
```java
@Getter@Setter
public class Product implements Serializable {
    private Long id;//商品id
    private String name;//商品名称
    private int price;//商品价格
    private int stock;//商品库存
}
```

### 7. 案例项目-商品服务product-server搭建

**步骤：**

- 使用Spring Initializr创建SpringBoot项目，选择Cloud Discover->Eureka Discover 和 Web->Web还有Lombok

- 把application.properties修改成application.yml，并添加配置信息
https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.0.RELEASE/reference/html/#netflix-eureka-client-starter

- 启动测试，会在Eureka注册中心控制台页面中看到product-server实例

- 启动多个实例，在注册中心管控台页面也可以看到.（在idea启动配置中添加-Dserver.port=8082参数，可以覆盖配置文件中的配置）

- 添加mapper,service,controller类.

***application.yml：**
```yml
server:
  port: 8081
spring:
  application:
    name: product-server
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

**ProductMapper：**
```java
// 商品服务业务代码，自己模拟数据，包括查询和获取的功能
@Component
public class ProductMapper {
    private Map<Long,Product> productMap = new HashMap();
    public ProductMapper(){
        Product p1 = new Product(1L,"小米手机",2799,10);
        Product p2 = new Product(2L,"苹果手机",5799,20);
        Product p3 = new Product(3L,"华为手机",4699,10);
        Product p4 = new Product(4L,"三星手机",4799,10);
        Product p5 = new Product(5L,"OPPO手机",3799,10);
        productMap.put(p1.getId(),p1);
        productMap.put(p2.getId(),p2);
        productMap.put(p3.getId(),p3);
        productMap.put(p4.getId(),p4);
        productMap.put(p5.getId(),p5);
    }
    public List<Product> list(){
        return new ArrayList<>(productMap.values());
    }
    public Product get(Long id){
        return productMap.get(id);
    }
}
```

**IProductService:**
```java
public interface IProductService {
    List<Product> list();
    Product get(Long id);
}
```

**ProductServiceImpl:**
```java
@Service
public class ProductServiceImpl implements IProductService {
    @Autowired
    private ProductMapper productMapper;
    @Override
    public List<Product> list() {
        return productMapper.list();
    }

    @Override
    public Product get(Long id) {
        return productMapper.get(id);
    }
}
```

**ProductController:**
```java
@RestController
@Slf4j
public class ProductController {
    @Autowired
    private IProductService productService;
    @Value("${server.port}")
    private String port;
 
   @RequestMapping("api/v1/product/find")  // 访问路径
    public Product find(Long id) {
        log.info("查找商品");
        Product product = productService.get(id);
        Product result = new Product();
        BeanUtils.copyProperties(product,result);
        result.setName(result.getName()+",data from "+port);
        System.out.println("进入....."+port);
        return result;
    }
}
```

### 8. SpringCloud Eureka 自我保护机制(了解)

Eureka Server 在运行期间会去统计心跳失败比例在 15 分钟之内是否低于 85%，如果低于 85%，Eureka Server 会将这些实例保护起来，让这些实例不会过期，但是在保护期内如果服务刚好这个服务提供者非正常下线了，此时服务消费者就会拿到一个无效的服务实例，此时会调用失败，对于这个问题需要服务消费者端要有一些容错机制，如重试，断路器等。

我们在单机测试的时候很容易满足心跳失败比例在 15 分钟之内低于 85%，这个时候就会触发 Eureka 的保护机制，一旦开启了保护机制，则服务注册中心维护的服务实例就不是那么准确了，此时我们可以使用
eureka.server.enable-self-preservation=false来关闭保护机制，这样可以确保注册中心中不可用的实例被及时的剔除（不推荐）

所以日常工作中设置为:
```yml
eureka.server.enable-self-preservation=true
```
![QQ图片20211008113104.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211008113104-ca671fceb40d4bfdb9f6638783a1d006.png)

## 三、微服务调用方式与负载均衡之 Ribbon

### 什么是 `RestTemplate`?

不是讲 `Ribbon` 么？怎么扯到了 `RestTemplate` 了？你先别急，听我慢慢道来。

我就说一句！**`RestTemplate`是`Spring`提供的一个访问Http服务的客户端类**，怎么说呢？就是微服务之间的调用是使用的 `RestTemplate` 。比如这个时候我们 消费者B 需要调用 提供者A 所提供的服务我们就需要这么写。如我下面的伪代码。

```java
@Autowired
private RestTemplate restTemplate;
// 这里是提供者A的ip地址，但是如果使用了 Eureka 那么就应该是提供者A的名称
private static final String SERVICE_PROVIDER_A = "http://localhost:8081";

@PostMapping("/judge")
public boolean judge(@RequestBody Request request) {
    String url = SERVICE_PROVIDER_A + "/service1";
    return restTemplate.postForObject(url, request, Boolean.class);
}
```
 如果你对源码感兴趣的话，你会发现上面我们所讲的 `Eureka` 框架中的 **注册**、**续约** 等，底层都是使用的 `RestTemplate` 。

### 为什么需要 Ribbon？

`Ribbon`  是 `Netflix` 公司的一个开源的负载均衡 项目，是一个客户端/进程内负载均衡器，**运行在消费者端**。

我们再举个:chestnut:，比如我们设计了一个秒杀系统，但是为了整个系统的 **高可用** ，我们需要将这个系统做一个集群，而这个时候我们消费者就可以拥有多个秒杀系统的调用途径了，如下图。

![秒杀系统ribbon.jpg](/upload/2021/10/%E7%A7%92%E6%9D%80%E7%B3%BB%E7%BB%9F-ribbon-b1c4b9ea2bdc4d3a9efc91ee23318f72.jpg)

如果这个时候我们没有进行一些 **均衡操作** ，如果我们对 `秒杀系统1` 进行大量的调用，而另外两个基本不请求，就会导致 `秒杀系统1` 崩溃，而另外两个就变成了傀儡，那么我们为什么还要做集群，我们高可用体现的意义又在哪呢？

所以 `Ribbon` 出现了，注意我们上面加粗的几个字——**运行在消费者端**。指的是，`Ribbon` 是运行在消费者端的负载均衡器，如下图。

![秒杀系统ribbon2.jpg](/upload/2021/10/%E7%A7%92%E6%9D%80%E7%B3%BB%E7%BB%9F-ribbon2-7878e0a465094d51859ef47c24190e4e.jpg)

其工作原理就是 `Consumer` 端获取到了所有的服务列表之后，在其**内部**使用**负载均衡算法**，进行对多个系统的调用。

### 案例项目-订单服务order-server搭建（没有分api,重要）

**步骤:**
- 使用Spring Initializr创建SpringBoot项目，选择Cloud Discover->Eureka Discover , Web->Web , Cloud Routing->Robbin
 
- 在项目的pom.xml文件添加product-api的依赖

- 添加相关的依赖配置

- 把创建订单的功能实现(获取商品信息暂先放下)

**application.yml:**
```yml
server:
  port: 8090
spring:
  application:
    name: order-server
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/  # 注册中心地址
```

**java类-Order**
```java
// 创建domain对象 Order类
@Data
public class Order implements Serializable {
    private String orderNo;//订单编号
    private Date createTime;//下单时间
    private String productName;//产品名称
    private int productPrice;//产品价格
    private Long userId;//用户id
}
```

**java类-IOrderService**
```java
public interface IOrderService {
	public Order save(Long userId, Long productId);
}
```

**java类-OrderServiceImpl**
```java
@Service
public class OrderServiceImpl implements IOrderService {
	@Autowired
	private RestTemplate restTemplate;

    @Override
    public Order save(Long userId, Long productId) {
        //  商品信息应该通过productId 从商品服务查询到！ 
	//  http请求：http://localhost:8081/api/v1/product/find?id=1  
	// 发送http请求的方式：如httpClient、RestTemplate（java中使用这个）等等
        Product product = restTemplate.getForObject("http://localhost:8081/api/v1/product/find?id="+productId, Product.class); // 远程获取
        Order order = new Order();
        order.setOrderNo(UUID.randomUUID().toString().replace("-",""));
        order.setCreateTime(new Date());
        order.setUserId(userId);
        order.setProductName(product.getName());
        order.setProductPrice(product.getPrice());
        System.out.println("执行保存订单操作");
        return order;
    }
}
```

**java类-OrderController**
```java
@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {
    @Autowired
    private IOrderService orderService;
    @RequestMapping
    public Order save(Long userId,Long productId){
        return orderService.save(userId,productId);
    }
}
```

**总结：Ribbon如何来实现远程调用?**

1. 使用restTemplate.getObject获取远程接口的信息.

2. 由于IoC容器中没有RestTemplate这个类，在配置文件中（即启动类中）添加RestTemplate的bean
```java
@Bean
public RestTemplate restTemplate() {
	return new RestTemplate();
}
```

3. 不在需要写具体的IP地址, 只需要写上服务的id--eureka中的service-id

> **这种方式有点low！写死了商品服务的url**

**总结：Ribbon如何来实现负载均衡？**

> 客户端负载均衡：Ribbon、Dubbo，服务端负载均衡：Nginx
> 如何判断?
> 调用者是否知道服务提供者的所有的地址信息，如果知道，就可以使用客户端负载，如果不知道就使用服务端负载。

**上面的远程调用存在问题：** 商品服务的url已经写死在了java代码中，若之后动态扩容后，是无法访问其他url的商品服务的！

> 请问如何做到多台商品服务器部署后的负载均衡？

**解决：**

1. 请求地址不写具体ip和端口，而是使用远程服务id(PRODUCT-SERVER为在注册中心注册的商品服务地址信息)

```java
Product product = restTemplate.getForObject("http://PRODUCT-SERVER/api/v1/product/find?id="+productId, Product.class); // 远程获取
```

2. 在RestTemplate的bean上贴上@LoadBalanced注解
```java
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
	return new RestTemplate();
}
```
![QQ图片20211008145024.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211008145024-999f5e8bc7924ba2b49f7d222aee8878.png)

### Nginx 和 Ribbon 的对比

提到 **负载均衡** 就不得不提到大名鼎鼎的 `Nignx` 了，而和 `Ribbon` 不同的是，它是一种**集中式**的负载均衡器。

何为集中式呢？简单理解就是 **将所有请求都集中起来，然后再进行负载均衡**。如下图。

![nginxvsribbon1.jpg](/upload/2021/10/nginx-vs-ribbon1-8e510e7add884691b0f51efac763d2bd.jpg)

我们可以看到 `Nginx` 是接收了所有的请求进行负载均衡的，而对于 `Ribbon` 来说它是在消费者端进行的负载均衡。如下图。

![nginxvsribbon2.jpg](/upload/2021/10/nginx-vs-ribbon2-d619847a7a8e4af88995654fd8f361bd.jpg)

> 请注意 `Request` 的位置，在 `Nginx` 中请求是先进入负载均衡器，而在 `Ribbon` 中是先在客户端进行负载均衡才进行请求的。

### Ribbon 的几种负载均衡算法

负载均衡，不管 `Nginx` 还是 `Ribbon` 都需要其算法的支持，如果我没记错的话 `Nginx` 使用的是 轮询和加权轮询算法。而在 `Ribbon` 中有更多的负载均衡调度算法，其默认是使用的 `RoundRobinRule` 轮询策略。

* **`RoundRobinRule`**：轮询策略。`Ribbon` 默认采用的策略。若经过一轮轮询没有找到可用的 `provider`，其最多轮询 10 轮。若最终还没有找到，则返回 `null`。
* **`RandomRule`**: 随机策略，从所有可用的 `provider` 中随机选择一个。
* **`RetryRule`**: 重试策略。先按照 `RoundRobinRule` 策略获取 `provider`，若获取失败，则在指定的时限内重试。默认的时限为 500 毫秒。

🐦🐦🐦 还有很多，这里不一一举:chestnut:了，你最需要知道的是默认轮询算法，并且可以更换默认的负载均衡算法，只需要在配置文件中做出修改就行。

```yml
providerName:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

**注意:** 服务的名称需要和代码中的服务名称一致，不然是修改不了负载均衡策略. RoundRobinRule这里为轮询算法规则

当然，在 `Ribbon` 中你还可以**自定义负载均衡算法**，你只需要实现 `IRule` 接口，然后修改配置文件或者自定义 `Java Config` 类。

### Ribbon负载均衡源码分析

RestTemplate的发送请求流程：

1. 如果是多个商品服务集群，首先就是必须要选择其中一个可用的服务器（客户端负载），如果是localhost:8082

2. 选择完成后就在订单服务内替换对应的URL请求地址："http://localhost:8082/api/v1/product/find?id="productId

3. 最后才是发送http请求： RestTemplate 执行它的 doExcute方法，其实内部还是使用的HttpUrlConnection

**上述总结：** 在发送请求之前需要对请求进行拦截处理：主要实现的方式就是添加拦截器，核心注解就是@LoadBalanced（添加负载均衡算法：就是在商品服务器列表中选择一个可用的）


## 四、什么是 Open Feign，微服务调用方式Feign(常用）

有了 `Eureka`  ，`RestTemplate` ，`Ribbon`，  我们就可以愉快地进行服务间的调用了，但是使用 `RestTemplate` 还是不方便，我们每次都要进行这样的调用。

```java
@Autowired
private RestTemplate restTemplate;
// 这里是提供者A的ip地址，但是如果使用了 Eureka 那么就应该是提供者A的名称
private static final String SERVICE_PROVIDER_A = "http://localhost:8081";

@PostMapping("/judge")
public boolean judge(@RequestBody Request request) {
    String url = SERVICE_PROVIDER_A + "/service1";
    // 是不是太麻烦了？？？每次都要 url、请求、返回类型的 
    return restTemplate.postForObject(url, request, Boolean.class);
}
```

这样每次都调用 `RestRemplate` 的 `API` 是否太麻烦，我能不能像**调用原来代码一样进行各个服务间的调用呢？**

:bulb::bulb::bulb:聪明的小朋友肯定想到了，那就用 **映射** 呀，就像域名和IP地址的映射。我们可以将被调用的服务代码映射到消费者端，这样我们就可以 **“无缝开发” **啦。

>  `OpenFeign` 也是运行在消费者端的，使用 `Ribbon` 进行负载均衡，所以 `OpenFeign` 直接内置了 `Ribbon`。

### 改造项目使用Feign方式实现远程调用

**步骤：**

- 在product-api项目中添加openfeign依赖

- 在product-api项目中添加ProductFeignApi接口

- 在product-server项目中添加ProductFeignApi的实现类(本质上就是个Controller),注意要把之前的controller删除掉.

- 在order-server项目中的启动类上贴上@EnableFeignClients注解，其属性basePackages用来表明扫描哪个包下面的贴了@FeignClient注解 (启动feign远程调用)

- 把之前RestTemplate的远程调用替换成Feign方式调用即可 （注意product-api和order-server中包名的问题.）

```java
@Service
public class OrderServiceImpl implements IOrderService {
    @Autowired
    private ProductFeignApi productFeignApi; // 生成JDK动态代理对象

    @Override
    public Order save(Long userId, Long productId) {
        //  商品信息应该通过productId 从商品服务查询到！ 
	//  http请求：http://localhost:8081/api/v1/product/find?id=1  
	// 发送http请求的方式：如httpClient、RestTemplate（java中使用这个）等等
	// springCloud -- > http --> feign
        Product product = productFeignApi.find(productId); // 远程获取 (没有了硬编码了)
        Order order = new Order();
        order.setOrderNo(UUID.randomUUID().toString().replace("-",""));
        order.setCreateTime(new Date());
        order.setUserId(userId);
        order.setProductName(product.getName());
        order.setProductPrice(product.getPrice());
        System.out.println("执行保存订单操作");
        return order;
    }
}
```

**ProductFeignApi（product-api中）**

```java
/**
1. 使用@FeignClient标记为feign接口，并且name参数值是提供服务的service-id
2. 使用 @RequestMapping规范请求的url地址
3. @RequestParam("id") 确定请求的参数
*/

@FeignClient(name = "product-server")
public interface ProductFeignApi {
    @RequestMapping("/api/v1/product/find")
    Product find(@RequestParam("id") Long id);
}
```

**ProductFeignClient（在product-server中具体实现）**
```java
@RestController
public class ProductFeignClient implements ProductFeignApi {
    @Autowired
    private IProductService productService;
    @Value("${server.port}")
    private String port;
    @Override
    public Product find(Long id) {
        Product product = productService.get(id);
        Product result = new Product();
        BeanUtils.copyProperties(product,result);
        result.setName(result.getName()+",data from "+port);
        return result;
    }
}
```

**创建动态代理后的执行:**

![QQ图片20211008162736.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211008162736-2e8a51dfc9c7444fb0dc8941ef3688d2.png)

### Feign调用源码分析

- 在启动类上贴上@EnableFeignClients会扫描当前包及其子包下贴了@FeignClient的接口并创建jdk默认动态代理对象.

- 代理会把请求交给SynchronousMethodHandler的invoke（）方法处理

- 底层是通过FeignLoadBalancer来帮我们实现远程调用，本质还是使用了Robbin，我们在配置文件中配置Robbin的负载均衡策略依然是可以生效的.

- 底层是调用Client.java中的execute方法，最底层就是通过HttpURLConnection把我们发送远程调用请求（Ribbon也是）

### Feign超时时间设置
为什么会给我们的服务设置超时时间？ 在调用多个服务时，防止一个服务无法返回而影响整个系统

源码中默认options中配置的是6000毫秒，但是Feign默认加入了Hystrix,此时默认是1秒超时.
我们可以通过修改配置，修改默认超时时间.
```yml
feign:
  client:
    config:
      default:
        connectTimeout: 2000   #2s
        readTimeout: 2000   
```

### Feign超时重试次数设置
```yml
https://github.com/Netflix/ribbon/wiki/Getting-Started#the-properties-file-sample-clientproperties

# Max number of retries on the same server (excluding the first try)
sample-client.ribbon.MaxAutoRetries=0

# Max number of next servers to retry (excluding the first server)
sample-client.ribbon.MaxAutoRetriesNextServer=1

ribbon:
  MaxAutoRetries: 2  #在一个服务器的最大重试次数，不包括第一次
  MaxAutoRetriesNextServer: 1   # 重试之后寻找下一个服务器的次数  （所以通过上面的设置，总共请求了（1+2） * 2 = 6次）
  ConnectTimeout: 1000
  ReadTimeout: 1000

PRODUCT-SERVER:
  ribbon:
    # NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
    # 在当前的服务器 重试的次数(不包括第一次的请求)
    MaxAutoRetries: 0
    # 在其他可用服务器的重试(不包括第一次的请求)
    MaxAutoRetriesNextServer: 0
    #(2+1+(2*1))
    ConnectTimeout: 1000
    ReadTimeout: 1000
```

## 五、必不可少的 Hystrix (服务熔断与降级)

### 什么是 Hystrix之熔断和降级

> 在分布式环境中，不可避免地会有许多服务依赖项中的某些失败。Hystrix是一个库，可通过添加等待时间容限和容错逻辑来帮助您控制这些分布式服务之间的交互。Hystrix通过隔离服务之间的访问点，停止服务之间的级联故障并提供后备选项来实现此目的，所有这些都可以提高系统的整体弹性。

> 熔断：类似保险丝. 防止整个系统故障，保护自己和下游服务，即目的是当A服务模块中的某块程序出现故障后为了不影响其他客户端的请求而做出的及时回应。

> 降级：抛弃非核心业务，保障核心页面的正常运行。即目的是为了解决整体项目的压力，而牺牲掉某一服务模块而采取的措施。

> 相同点：1)目的很一致: 都是从可用性可靠性着想，为防止系统的整体缓慢甚至崩溃，采用的技术手段；
2)最终表现类似: 对于两者来说，最终让用户体验到的是某些功能暂时不可达或不可用；
>
> 不同点：触发原因不太一样:服务熔断一般是某个服务（下游服务）故障引起，而服务降级一般是从整体负荷考虑；

总体来说 `Hystrix` 就是一个能进行 **熔断** 和 **降级** 的库，通过使用它能提高整个系统的弹性。

那么什么是 熔断和降级 呢？再举个:chestnut:，此时我们整个微服务系统是这样的。服务A调用了服务B，服务B再调用了服务C，但是因为某些原因，服务C顶不住了，这个时候大量请求会在服务C阻塞。

![Hystrix1.jpg](/upload/2021/10/Hystrix1-a3e7ecb6d0324023baf5c6993989bb99.jpg)

服务C阻塞了还好，毕竟只是一个系统崩溃了。但是请注意这个时候因为服务C不能返回响应，那么服务B调用服务C的的请求就会阻塞，同理服务B阻塞了，那么服务A也会阻塞崩溃。

> 请注意，为什么阻塞会崩溃。因为这些请求会消耗占用系统的线程、IO 等资源，消耗完你这个系统服务器不就崩了么。

![Hystrix2.jpg](/upload/2021/10/Hystrix2-63403a2d92e24a9b9cd12391625715a5.jpg)<img src="https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/Hystrix2.jpg" style="zoom:50%;" />

这就叫 **服务雪崩**。妈耶，上面两个 **熔断** 和 **降级** 你都没给我解释清楚，你现在又给我扯什么 **服务雪崩** ？:tired_face::tired_face::tired_face:

### 服务雪崩效应原因和解决思路(重要)
商品详情展示服务会依赖商品服务，价格服务，商品评论服务，调用三个依赖服务会共享商品详情服务的线程池，如果其中的商品评论服务不可用（超时，代码异常等等）, 就会出现线程池里所有线程都因等待响应而被阻塞, 从而造成服务雪崩。
概括来说就是大量请求线程同步等待造成的资源耗尽。

**解决方案：**

1. 超时机制：如果我们加入超时机制，例如2s，那么超过2s就会直接返回了，那么这样就在一定程度上可以抑制消费者资源耗尽的问题。

2. 服务限流：通过线程池+队列的方式或者通过信号量的方式。比如商品评论比较慢，最大能同时处理10个线程，队列待处理5个，那么如果同时20个线程到达的话，其中就有5个线程被限流了，其中10个先被执行，另外5个在队列中（在zull、nginx常用）

3. 服务熔断：当依赖的服务有大量超时时，在让新的请求去访问根本没有意义，只会无畏的消耗现有资源，比如我们设置了超时时间为1s，如果短时间内有大量请求在1s内都得不到响应，就意味着这个服务出现了异常，此时就没有必要再让其他的请求去访问这个服务了，这个时候就应该使用熔断器避免资源浪费。即当指定时间窗内的请求失败率达到设定阈值时，系统将通过 **断路器** 直接将此请求链路断开。

4. 服务降级：有服务熔断，必然要有服务降级。所谓降级，就是当某个服务熔断之后，服务将不再被调用，此时客户端可以自己准备一个本地的fallback（回退）回调，返回一个缺省值。 例如：(备用接口/缓存/mock数据)，这样做，虽然服务水平下降，但好歹可用，比直接挂掉要强，当然这也要看适合的业务场景。

5. 资源隔离：也称熔断隔离策略，如线程隔离（后面细说）

### 具体实现Hystrix简介(了解)

**Hystrix是什么？** 在分布式环境中，许多服务依赖中的一些不可避免地会失败。Hystrix 是一个库，它通过添加延迟容错和容错逻辑来帮助您控制这些分布式服务之间的交互。Hystrix 通过隔离服务之间的访问点、阻止它们之间的级联故障并提供回退选项来实现这一点，所有这些都提高了系统的整体弹性。

**为啥要用Hystrix？** 在大中型分布式系统中，通常系统很多依赖(HTTP,hession,Netty,Dubbo等)，在高并发访问下,这些依赖的稳定性与否对系统的影响非常大,但是依赖有很多不可控问题:如网络连接缓慢，资源繁忙，暂时不可用，服务脱机等。当依赖阻塞时,大多数服务器的线程池就出现阻塞(BLOCK),影响整个线上服务的稳定性，在复杂的分布式架构的应用程序有很多的依赖，都会不可避免地在某些时候失败。高并发的依赖失败时如果没有隔离措施，当前应用服务就有被拖垮的风险。

** Hystrix能做啥？** 提供了熔断、隔离、Fallback、cache、监控等功能：
1. 通过第三方客户端库访问（通常通过网络）依赖项，提供对延迟和故障的保护和控制。
2. 停止复杂分布式系统中的级联故障。
3. 快速失败并快速恢复。
4. 在可能的情况下回退并优雅地降级。
5. 实现近乎实时的监控、警报和操作控制。

**Hystrix 背后的设计原则是什么？工作原理？**
1. 防止任何单个依赖项耗尽所有容器（例如 Tomcat）用户线程。
2. 减轻负载和快速失败而不是排队。
3. 在可行的情况下提供回退以保护用户免于失败。
4. 使用隔离技术（例如隔板、泳道和断路器模式）来限制任何一种依赖性的影响。
5. 通过近乎实时的指标、监控和警报优化发现时间
6. 通过配置更改的低延迟传播和支持 Hystrix 大部分方面的动态属性更改来优化恢复时间，这允许您使用低延迟反馈循环进行实时操作修改。
7. 防止整个依赖客户端执行中的故障，而不仅仅是网络流量。

**Hystrix 如何实现其目标？**

1. 将所有对外部系统（或“依赖项”）的调用包装在一个HystrixCommandorHystrixObservableCommand对象中，该对象通常在单独的线程中执行（这是命令模式的一个示例）。
2. 超时调用时间超过您定义的阈值。有一个默认值，但对于大多数依赖项，您可以通过“属性”自定义设置这些超时，以便它们略高于每个依赖项的测量的 99.5 th百分位性能。
3. 为每个依赖项维护一个小的线程池（或信号量）；如果它已满，则发往该依赖项的请求将立即被拒绝，而不是排队。
4. 测量成功、失败（客户端抛出的异常）、超时和线程拒绝。
5. 如果服务的错误百分比超过阈值，则手动或自动触发断路器以在一段时间内停止对特定服务的所有请求。
6. 当请求失败、被拒绝、超时或短路时执行回退逻辑。
7. 近乎实时地监控指标和配置更改。

### order-server项目集成Hystrix(重要)？

以下是在测试服务降级：原业务方法与降级之后的方法都调用了。而熔断是原业务方法是不会调用的！

01.在order-server中添加hystrix依赖

02.在启动类中添加@EnableCircuitBreaker注解，开启熔断策略

03.在最外层添加熔断降级的处理. 在order-server中的控制器中添加@HystrixCommand(fallbackMethod = "saveFail")注解   （注意fallbackMethod后面的参数方法名就是降级后才调用的方法，且该方法需要和原方法一样的签名，） （在需要熔断的控制器上加注解）

**而开启熔断需要三个条件（默认情况下）：**
1. 在指定的时间的时间窗口。如10s
2. 需要达到一定的请求量，如20个请求
3. 需要请求的错误率达到50%以上
总结就是在10s之内，发送了20个请求，而且100个请求的错误率达到了50%，才有机会去触发服务熔断。上述属性可以改


### 熔断降级服务异常报警通知(了解)？配置redis相关配置
```java
public Order saveFail(Long userId,Long productId){
        new Thread(()->{
            String redisKey = "order-save";
            String value = stringRedisTemplate.opsForValue().get(redisKey);
            if(StringUtils.isEmpty(value)){
                System.out.println("order下订单服务失败，请查找原因.");
                stringRedisTemplate.opsForValue().set(redisKey,"save-order-fail",20, TimeUnit.SECONDS);
            }else{
                System.out.println("已经发送过短信");
            }
        }).start();
        return new Order();
    }
```

### 重点属性讲解(重要)
```java
            commandProperties = {@HystrixProperty(name="circuitBreaker.requestVolumeThreshold",value = "100"),//配置请求量
                    @HystrixProperty(name="metrics.rollingStats.timeInMilliseconds",value = "3000"),//配置时间窗口
                    @HystrixProperty(name="circuitBreaker.errorThresholdPercentage",value = "50"),//配置出现错误百分比
                    @HystrixProperty(name="execution.isolation.semaphore.maxConcurrentRequests",value = "30"),//配置最大请求连接数量
                    @HystrixProperty(name="fallback.isolation.semaphore.maxConcurrentRequests",value = "1000"),//配置降级处理的最大并发数
                    @HystrixProperty(name="execution.isolation.strategy",value = "semaphore")//配置处理模式, 信号量的模式还是线程池的模式
            },
            threadPoolProperties = {@HystrixProperty(name="coreSize",value = "10")})//对于线程模式, 配置最大的核心处理线程
```

**熔断隔离策略: 线程池(默认)与信号量**

线程池：服务耗时比较长的建议使用线程池
信号量：服务响应很快的建议使用信号量.比如访问的缓存服务等.

![QQ图片20211008194600.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211008194600-ed30a4e954644269ad6e1a2aadc8f98e.png)

**超时时间调整:**
```yml
#是否开启超时限制 （一定不要禁用）
hystrix:
	  command:
	    default:
			execution:
				timeout:
					enabled: 

#超时时间调整
hystrix:
	command:
		default:
	      execution:
	        isolation:
	          thread:
	            timeoutInMilliseconds: 4000
```

### 断路器Dashboard监控仪表盘(了解)

它主要用来实时监控Hystrix的各项指标信息。通过Hystrix Dashboard反馈的实时信息，可以帮助我们快速发现系统中存在的问题。


### 例子
其实这里所讲的 **熔断** 就是指的 `Hystrix` 中的 **断路器模式** ，你可以使用简单的 `@HystrixCommand` 注解来标注某个方法，这样 `Hystrix` 就会使用 **断路器** 来“包装”这个方法，每当调用时间超过指定时间时(默认为1000ms)，断路器将会中断对这个方法的调用。

当然你可以对这个注解的很多属性进行设置，比如设置超时时间，像这样。

```java
@HystrixCommand(
    commandProperties = {@HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "1200")}
)
public List<Xxx> getXxxx() {
    // ...省略代码逻辑
}
```

但是，我查阅了一些博客，发现他们都将 **熔断** 和 **降级** 的概念混淆了，以我的理解，**降级是为了更好的用户体验，当一个方法调用异常时，通过执行另一种代码逻辑来给用户友好的回复**。这也就对应着 `Hystrix` 的 **后备处理** 模式。你可以通过设置 `fallbackMethod` 来给一个方法设置备用的代码逻辑。比如这个时候有一个热点新闻出现了，我们会推荐给用户查看详情，然后用户会通过id去查询新闻的详情，但是因为这条新闻太火了(比如最近什么*易对吧)，大量用户同时访问可能会导致系统崩溃，那么我们就进行 **服务降级** ，一些请求会做一些降级处理比如当前人数太多请稍后查看等等。

```java
// 指定了后备方法调用
@HystrixCommand(fallbackMethod = "getHystrixNews")
@GetMapping("/get/news")
public News getNews(@PathVariable("id") int id) {
    // 调用新闻系统的获取新闻api 代码逻辑省略
}
// 
public News getHystrixNews(@PathVariable("id") int id) {
    // 做服务降级
    // 返回当前人数太多，请稍后查看
}
```

### 什么是Hystrix之其他

我在阅读 《Spring微服务实战》这本书的时候还接触到了一个 **舱壁模式** 的概念。在不使用舱壁模式的情况下，服务A调用服务B，这种调用默认的是 **使用同一批线程来执行** 的，而在一个服务出现性能问题的时候，就会出现所有线程被刷爆并等待处理工作，同时阻塞新请求，最终导致程序崩溃。而舱壁模式会将远程资源调用隔离在他们自己的线程池中，以便可以控制单个表现不佳的服务，而不会使该程序崩溃。

具体其原理我推荐大家自己去了解一下，本篇文章中对 **舱壁模式** 不做过多解释。当然还有 **`Hystrix` 仪表盘**，它是**用来实时监控 `Hystrix` 的各项指标信息的**，这里我将这个问题也抛出去，希望有不了解的可以自己去搜索一下。

## 六、微服务网关——Zuul

> ZUUL 是从设备和 web 站点到 Netflix 流应用后端的所有请求的前门。作为边界服务应用，ZUUL 是为了实现动态路由、监视、弹性和安全性而构建的。它还具有根据情况将请求路由到多个 Amazon Auto Scaling Groups（亚马逊自动缩放组，亚马逊的一种云计算方式） 的能力
>
> Zuul网关是系统的唯一对外的入口，介于客户端和服务器端之间的中间层，处理非业务功能提供路由请求、鉴权、监控、缓存、限流等功能


![QQ图片20211008204633.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211008204633-2664ddd6c2fc4850ad030ea2de47d64f.png)

### zuul网关作用？
除了上图之外，还有：

1. **验证与安全保障:** 识别面向各类资源的验证要求并拒绝那些与要求不符的请求。
2. **审查与监控:** 在边缘位置追踪有意义数据及统计结果，从而为我们带来准确的生产状态结论。
3. **动态路由:** 以动态方式根据需要将请求路由至不同后端集群处。
4. **压力测试:** 逐渐增加指向集群的负载流量，从而计算性能水平。
5. **负载分配:** 为每一种负载类型分配对应容量，并弃用超出限定值的请求。
6. **静态响应处理:** 在边缘位置直接建立部分响应，从而避免其流入内部集群。
7. **多区域弹性:** 跨越AWS区域进行请求路由，旨在实现ELB使用多样化并保证边缘位置与使用者尽可能接近。

### 网关项目zuul-server搭建

**步骤：**
- 使用Spring Initializr创建SpringBoot项目，选择Cloud Discover->Eureka Discover , Cloud Rounting -> Zuul

- 添加application.yml配置文件并添加相关的配置信息.
```yml
# 网关也会在注册中心注册服务
server:
  port: 9000
spring:
  application:
    name: zuul-server
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

- 在启动类zuulApplication上贴上@EnableZuulProxy注解


在上面我们学习了 `Eureka` 之后我们知道了 *服务提供者*  是 *消费者* 通过 `Eureka Server` 进行访问的，即 `Eureka Server` 是 *服务提供者* 的统一入口。那么整个应用中存在那么多 *消费者* 需要用户进行调用，这个时候用户该怎样访问这些 *消费者工程* 呢？当然可以像之前那样直接访问这些工程。但这种方式没有统一的消费者工程调用入口，不便于访问与管理，而 Zuul 就是这样的一个对于 *消费者* 的统一入口。

> 如果学过前端的肯定都知道 Router 吧，比如 Flutter 中的路由，Vue，React中的路由，用了 Zuul 你会发现在路由功能方面和前端配置路由基本是一个理。:smile: 我偶尔撸撸 Flutter。

大家对网关应该很熟吧，简单来讲网关是系统唯一对外的入口，介于客户端与服务器端之间，用于对请求进行**鉴权**、**限流**、 **路由**、**监控**等功能。

![zuulsj22o93nfdsjkdsf.jpg](/upload/2021/10/zuul-sj22o93nfdsjkdsf-3162e638b0804e6ab760b2185d3290e9.jpg)

没错，网关有的功能，`Zuul` 基本都有。而 `Zuul` 中最关键的就是 **路由和过滤器** 了，在官方文档中 `Zuul` 的标题就是

> Router and Filter : Zuul

### Zuul 的路由功能

#### 简单配置

本来想给你们复制一些代码，但是想了想，因为各个代码配置比较零散，看起来也比较零散，我决定还是给你们画个图来解释吧。

比如这个时候我们已经向 `Eureka Server` 注册了两个 `Consumer` 、三个 `Provicer` ，这个时候我们再加个 `Zuul` 网关应该变成这样子了。

![zuulsj22o93nfdsjkdsf2312.jpg](/upload/2021/10/zuul-sj22o93nfdsjkdsf2312-03587580f6044010b900df55e11f360c.jpg)

emmm，信息量有点大，我来解释一下。关于前面的知识我就不解释了:neutral_face:。

首先，`Zuul` 需要向 `Eureka` 进行注册，注册有啥好处呢？

你傻呀，`Consumer` 都向 `Eureka Server` 进行注册了，我网关是不是只要注册就能拿到所有 `Consumer` 的信息了？

拿到信息有什么好处呢？

我拿到信息我是不是可以获取所有的 `Consumer` 的元数据(名称，ip，端口)？

拿到这些元数据有什么好处呢？拿到了我们是不是直接可以做**路由映射**？比如原来用户调用 `Consumer1` 的接口 `localhost:8001/studentInfo/update` 这个请求，我们是不是可以这样进行调用了呢？`localhost:9000/consumer1/studentInfo/update` 呢？你这样是不是恍然大悟了？

> 这里的url为了让更多人看懂所以没有使用 restful 风格。

上面的你理解了，那么就能理解关于 `Zuul` 最基本的配置了，看下面。

```yml
server:
  port: 9000
eureka:
  client:
    service-url:
      # 这里只要注册 Eureka 就行了
      defaultZone: http://localhost:9997/eureka
```

然后在启动类上加入 `@EnableZuulProxy` 注解就行了。没错，就是那么简单:smiley:。

#### 统一前缀

这个很简单，就是我们可以在前面加一个统一的前缀，比如我们刚刚调用的是 `localhost:9000/consumer1/studentInfo/update`，这个时候我们在 `yaml` 配置文件中添加如下。

```yml
zuul:
  prefix: /zuul
```

这样我们就需要通过 `localhost:9000/zuul/consumer1/studentInfo/update` 来进行访问了。

#### 路由策略配置

你会发现前面的访问方式(直接使用服务名)，需要将微服务名称暴露给用户，会存在安全性问题。所以，可以自定义路径来替代微服务名称，即自定义路由策略。

```yml
zuul:
  routes:
    consumer1: /FrancisQ1/**
    consumer2: /FrancisQ2/**
```

这个时候你就可以使用 ` `localhost:9000/zuul/FrancisQ1/studentInfo/update` 进行访问了。

#### 服务名屏蔽

这个时候你别以为你好了，你可以试试，在你配置完路由策略之后使用微服务名称还是可以访问的，这个时候你需要将服务名屏蔽。

```yml
zuul:
  ignore-services: "*"
```

#### 路径屏蔽

`Zuul` 还可以指定屏蔽掉的路径 URI，即只要用户请求中包含指定的 URI 路径，那么该请求将无法访问到指定的服务。通过该方式可以限制用户的权限。

```yml
zuul:
  ignore-patterns: **/auto/**
```

这样关于 auto 的请求我们就可以过滤掉了。

> ** 代表匹配多级任意路径
>
> *代表匹配一级任意路径

#### 敏感请求头屏蔽

默认情况下，像 `Cookie`、`Set-Cookie` 等敏感请求头信息会被 `zuul` 屏蔽掉，我们可以将这些默认屏蔽去掉，当然，也可以添加要屏蔽的请求头。

总的来说，定义路由规则主要有两种：

1. 基于URL进行配置：
```yml
zuul:
  ignoredPatterns: /*-server/**
  routes:
    order-server-route:
      path: /product/**    # 请求的地址product下面的所有
      url: http://localhost:8081/    # 经过网关后访问这个地址，这个基于URL配置的只能访问固定的一个商品服务，若想访问多个，要使用基于服务名称的配置
```

2. 基于服务名称进行配置：
```yml
zuul:
  ignoredPatterns: /*-server/**  #忽略地址
  routes:
    order-server-route: #路由Id, 自定义名字, 不能重复
      path: /order/**     #访问路径,  ? 代表单个字符, * 代表多个字符, 但是不能跨层级, ** 代表多个字符, 可以跨层级
      serviceId: order-server  # 服务的ID 注册中心注册的serviceId
```

### zuul流程分析

https://github.com/Netflix/zuul/wiki/How-it-Works

底层是一个ZuulServlet

装配开关: 满足一定的条件才去装配对应的Bean对象

```
本质上是经过一系列的过滤器
过滤器的关联，主要就是通过一个ZuulServlet
其流程：ZuulServlet - > ZuulServlet #service() - ->  service() #preRoute() --> service() #route() --> service() #postRoute() 
```
![687474703a2f2f6e6574666c69782e6769746875622e696f2f7a75756c2f696d616765732f7a75756c2d726571756573742d6c6966656379636c652e706e67.png](/upload/2021/10/687474703a2f2f6e6574666c69782e6769746875622e696f2f7a75756c2f696d616765732f7a75756c2d726571756573742d6c6966656379636c652e706e67-42c4f66e97a64a6a99a1de1490de6ee7.png)


### Zuul 的过滤功能 

如果说，路由功能是 `Zuul` 的基操的话，那么**过滤器**就是 `Zuul`的利器了。毕竟所有请求都经过网关(Zuul)，那么我们可以进行各种过滤，这样我们就能实现 **限流**，**灰度发布**，**权限控制** 等等。

#### 简单实现一个请求时间日志打印

要实现自己定义的 `Filter` 我们只需要继承 `ZuulFilter` 然后将这个过滤器类以 `@Component` 注解加入 Spring 容器中就行了。

在给你们看代码之前我先给你们解释一下关于过滤器的一些注意点。

![687474703a2f2f6e6574666c69782e6769746875622e696f2f7a75756c2f696d616765732f7a75756c2d726571756573742d6c6966656379636c652e706e67.png](/upload/2021/10/687474703a2f2f6e6574666c69782e6769746875622e696f2f7a75756c2f696d616765732f7a75756c2d726571756573742d6c6966656379636c652e706e67-b86dee4b10364f72b0e0b883ccf2a434.png)

过滤器类型：`Pre`、`Routing`、`Post`。前置`Pre`就是在请求之前进行过滤，`Routing`路由过滤器就是我们上面所讲的路由策略，而`Post`后置过滤器就是在 `Response` 之前进行过滤的过滤器。你可以观察上图结合着理解，并且下面我会给出相应的注释。

```java
// 自定义登录鉴权zuul过滤器，实现ZuulFilter
// 加入Spring容器
@Component
public class PreRequestFilter extends ZuulFilter {
    // 返回过滤器类型pre、route、post、error, 这里是前置过滤器
    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    // 指定过滤顺序 越小越先执行，这里第一个执行
    // 当然不是只真正第一个 在Zuul内置中有其他过滤器会先执行
    // 那是写死的 比如 SERVLET_DETECTION_FILTER_ORDER = -3
    @Override
    public int filterOrder() {
        return 0;
    }

    // 什么时候该进行过滤
    // 是否需要进行过滤，true：表示执行过滤业务 run方法
    // 这里我们可以进行一些判断，这样我们就可以过滤掉一些不符合规定的请求等等
    @Override
    public boolean shouldFilter() {
        RequestContext requestContext = RequestContext.getCurrentContext();
        HttpServletRequest request = requestContext.getRequest();
        if(request.getRequestURI().indexOf("/order/api/")>=0){
            return true;
        }
        return false;
    }

    // 如果过滤器允许通过则怎么进行处理，即具体的业务处理逻辑
    @Override
    public Object run() throws ZuulException {
        RequestContext requestContext = RequestContext.getCurrentContext();
        HttpServletRequest request = requestContext.getRequest();  // 获取到请求对象request对象
        String cookie = request.getHeader("Cookie");
        if(StringUtils.isEmpty(cookie)){  // 判断请求参数Cookie
            cookie = request.getParameter("Cookie");
        }
        if(StringUtils.isEmpty(cookie)){ // 判断cookie是否为空
            requestContext.setSendZuulResponse(false);
            requestContext.setResponseStatusCode(HttpStatus.UNAUTHORIZED.value()); // 返回状态码
        }
        return null;
    }
}
```


```java
// lombok的日志
@Slf4j
// 加入 Spring 容器
@Component
public class AccessLogFilter extends ZuulFilter {
    // 指定该过滤器的过滤类型
    // 此时是后置过滤器
    @Override
    public String filterType() {
        return FilterConstants.POST_TYPE;
    }
    // SEND_RESPONSE_FILTER_ORDER 是最后一个过滤器
    // 我们此过滤器在它之前执行
    @Override
    public int filterOrder() {
        return FilterConstants.SEND_RESPONSE_FILTER_ORDER - 1;
    }
    @Override
    public boolean shouldFilter() {
        return true;
    }
    // 过滤时执行的策略
    @Override
    public Object run() throws ZuulException {
        RequestContext context = RequestContext.getCurrentContext();
        HttpServletRequest request = context.getRequest();
        // 从RequestContext获取原先的开始时间 并通过它计算整个时间间隔
        Long startTime = (Long) context.get("startTime");
        // 这里我可以获取HttpServletRequest来获取URI并且打印出来
        String uri = request.getRequestURI();
        long duration = System.currentTimeMillis() - startTime;
        log.info("uri: " + uri + ", duration: " + duration / 100 + "ms");
        return null;
    }
}
```

上面就简单实现了请求时间日志打印功能，你有没有感受到 `Zuul` 过滤功能的强大了呢？

没有？好的、那我们再来。

#### 令牌桶限流

当然不仅仅是令牌桶限流方式，`Zuul` 只要是限流的活它都能干，这里我只是简单举个:chestnut:。

<img src="https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/zuui-令牌桶限流.jpg" alt="令牌桶限流" style="zoom:50%;" />

我先来解释一下什么是 **令牌桶限流** 吧。

首先我们会有个桶，如果里面没有满那么就会以一定 **固定的速率** 会往里面放令牌，一个请求过来首先要从桶中获取令牌，如果没有获取到，那么这个请求就拒绝，如果获取到那么就放行。很简单吧，啊哈哈、

下面我们就通过 `Zuul` 的前置过滤器来实现一下令牌桶限流。

```java
package com.lgq.zuul.filter;

import com.google.common.util.concurrent.RateLimiter;
import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.exception.ZuulException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;
import org.springframework.stereotype.Component;

@Component
@Slf4j
public class RouteFilter extends ZuulFilter {
    // 定义一个令牌桶，每秒产生2个令牌，即每秒最多处理2个请求
    private static final RateLimiter RATE_LIMITER = RateLimiter.create(2);
    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return -5;
    }

    @Override
    public Object run() throws ZuulException {
        log.info("放行");
        return null;
    }

    @Override
    public boolean shouldFilter() {
        RequestContext context = RequestContext.getCurrentContext();
        if(!RATE_LIMITER.tryAcquire()) {
            log.warn("访问量超载");
            // 指定当前请求未通过过滤
            context.setSendZuulResponse(false);
            // 向客户端返回响应码429，请求数量过多
            context.setResponseStatusCode(429);
            return false;
        }
        return true;
    }
}
```

这样我们就能将请求数量控制在一秒两个，有没有觉得很酷？

### EnableZuulProxy和EnableZuulServer的区别？

zuul提供了两个注解 @EnableZuulProxy， @EnableZuulServer，来启用不同的过滤器集合。

@EnableZuulProxy 启用的过滤器 是@EnableZuulServer 的超集  
PreDecorationFilter: 代理的Filter
RibbonRoutingFilter: 基于ServiceId远程调用的filter
SimpleHostRoutingFilter: 基于Url的远程调用的filter
@EnableZuulServer 不提供路由功能，作为server模式而不是代理模式运行

Histrix熔断: 
Ribbon: 负载均衡远程调用

超时时间配置: 直接配置Ribbon的超时时间即可, 如果两个都配置的话, 取最小值

### 关于 Zuul  的其他

`Zuul` 的过滤器的功能肯定不止上面我所实现的两种，它还可以实现 **权限校验**，包括我上面提到的 **灰度发布** 等等。

当然，`Zuul` 作为网关肯定也存在 **单点问题** ，如果我们要保证 `Zuul` 的高可用，我们就需要进行 `Zuul` 的集群配置，这个时候可以借助额外的一些负载均衡器比如 `Nginx` 。

## 七、链路追踪组件Sleuth&Zipkin(了解)

### 为什么需要链路追踪？

微服务架构是一个分布式架构，它按业务划分服务单元，一个分布式系统往往有很多个服务单元。由于服务单元数量众多，业务的复杂性，如果出现了错误和异常，很难去定位。

主要体现在，一个请求可能需要调用很多个服务，而内部服务的调用复杂性，决定了问题难以定位。所以微服务架构中，必须实现分布式链路追踪，去跟进一个请求到底有哪些服务参与，参与的顺序又是怎样的，从而达到每个请求的步骤清晰可见，出了问题，很快定位。

在微服务系统中，一个来自用户的请求，请求先达到前端A（如前端界面），然后通过远程调用，达到系统的中间件B、C（如负载均衡、网关等），最后达到后端服务D、E，后端经过一系列的业务逻辑计算最后将数据返回给用户。对于这样一个请求，经历了这么多个服务，怎么样将它的请求过程的数据记录下来呢？这就需要用到服务链路追踪。

![QQ图片20211009222439.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211009222439-41a47597d94546eaab57adff15932a08.png)

### 什么叫埋点？

所谓埋点就是在应用中特定的流程收集一些信息，用来跟踪应用使用的状况，后续用来进一步优化产品或是提供运营的数据支撑，包括访问数（Visits），访客数（Visitor），停留时长（Time On Site），页面浏览数（Page Views）和跳出率（Bounce Rate）。这样的信息收集可以大致分为两种：页面统计（track this virtual page view），统计操作行为（track this button by an event）。

在京东、淘宝等浏览商品信息后，后台就会将数据请求到后台保存！

### 集成链路追踪组件Sleuth

[Sleuth是一个专门用于记录链路数据的开源组件](https://cloud.spring.io/spring-cloud-static/spring-cloud-sleuth/2.2.0.RELEASE/reference/html/)

**步骤：**
- 在product-server和order-server中添加sleuth依赖、

- 在需要写日志的类上贴@Slf4j,然后再order-server,product-server中打印日志。

> **Sleuth可以实现对该链路的唯一标记！**

**依赖：**
```java
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```

**日志格式：**
```
例如：[order-server,c323c72e7009c077,fba72d9c65745e60,false] 

1、第一个值，spring.application.name的值
			
2、第二个值（Trace ID），c323c72e7009c077 ，sleuth生成的一个ID，叫Trace ID，用来标识一条请求链路，一条请求链路中包含一个Trace ID（全局唯一性），多个Span ID
			
3、第三个值（Span ID），fba72d9c65745e60、spanID 基本的工作单元，获取元数据，如发送一个http

4、第四个值：true，是否要将该信息输出到zipkin服务中来收集和展示。
```

### 什么是Zipkin？

[zipkin](https://zipkin.io/) 是Twitter基于google的分布式监控系统Dapper（论文）的开发源实现，zipkin用于跟踪分布式服务之间的应用数据链路，分析处理延时，帮助我们改进系统的性能和定位故障。

其中同类产品包括：

1. CAT：由大众点评开源，基于Java开发的实时应用监控平台，包括实时应用监控，业务监控 。 耦合性高一定  需要自定义埋点信息。
2. Pinpoint：由韩国团队naver团队开源，针对大规模分布式系统用链路监控，使用Java写的工具。
3. SkyWalking：2015年由个人吴晟（华为开发者）开源 ， 2017年加入Apache孵化器。针对分布式系统的应用性能监控系统，特别针对微服务、cloud native和容器化(Docker, Kubernetes, Mesos)架构，其核心是个分布式追踪系统。

### Zipkin+Sleuth整合？

- 在微服务项目中添加zipkin依赖(zipkin依赖中已经包含sleuth,所以可以把之前的sleuth依赖删除)
```java
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```
- 启动zipkin服务,我们直接使用的是jar的方式.
- 需要在product-server和order-server中的配置文件中添加zipkin地址.
```yml
spring:
  zipkin:
    base-url: http://localhost:9411   # 往这里发日志请求
  sleuth:
    sampler:
      probability: 1   # 1表示所有的日志都会请求回去，一般设置0.1，即10%的日志会请求回去
```

## 八、Spring Cloud配置管理——Config

### 为什么要使用进行配置管理？

1. 当我们的微服务系统开始慢慢地庞大起来，那么多 `Consumer` 、`Provider` 、`Eureka Server` 、`Zuul` 系统都会持有自己的配置，这个时候我们在项目运行的时候可能需要更改某些应用的配置，如果我们不进行配置的统一管理，我们只能**去每个应用下一个一个寻找配置文件然后修改配置文件再重启应用**。

2. 统一管理配置, 快速切换各个环境的配置

3. 在微服务体系中，服务的数量以及配置信息的日益增多，比如各种服务器参数配置、各种数据库访问参数配置、各种环境下配置信息的不同、配置信息修改之后实时生效等等，传统的配置文件方式或者将配置信息存放于数据库中的方式已无法满足开发人员对配置管理的要求，如：

- **安全性：** 配置跟随源代码保存在代码库中，容易造成配置泄漏
- **时效性：** 修改配置，需要重启服务才能生效
- **局限性：** 无法支持动态调整：例如日志开关、功能开关


首先对于分布式系统而言我们就不应该去每个应用下去分别修改配置文件，再者对于重启应用来说，服务无法访问所以直接抛弃了可用性，这是我们更不愿见到的。

那么有没有一种方法**既能对配置文件统一地进行管理，又能在项目运行时动态修改配置文件呢？**

那就是我今天所要介绍的 `Spring Cloud Config` 。

> 能进行配置管理的框架不止 `Spring Cloud Config` 一种，大家可以根据需求自己选择（百度的disconf，阿里的diamand，携程阿波罗：Apollo（最强大）等等）。而且对于 `Config` 来说有些地方实现的不是那么尽人意。

### Config 是什么

> `Spring Cloud Config` 为分布式系统中的外部化配置提供服务器和客户端支持。使用 `Config` 服务器，可以在中心位置管理所有环境中应用程序的外部属性。

简单来说，`Spring Cloud Config` 就是能将各个 应用/系统/模块 的配置文件存放到 **统一的地方然后进行管理**(Git 或者 SVN)。

**为什么要使用配置中心？ 其大致工作流程：**

![QQ图片20211010125619.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211010125619-9a65a9ae2f194f95ac777760900de2bd.png)

你想一下，我们的应用是不是只有启动的时候才会进行配置文件的加载，那么我们的 `Spring Cloud Config` 就暴露出一个接口给启动应用来获取它所想要的配置文件，应用获取到配置文件然后再进行它的初始化工作。就如下图。

![configksksks.jpg](/upload/2021/10/config-ksksks-fe25cdaef9674cc283209f90e16bf6b7.jpg)

**当然这里你肯定还会有一个疑问，如果我在应用运行时去更改远程配置仓库(Git)中的对应配置文件，那么依赖于这个配置文件的已启动的应用会不会进行其相应配置的更改呢？**

答案是不会的。

什么？那怎么进行动态修改配置文件呢？这不是出现了 **配置漂移** 吗？你个渣男:rage:，你又骗我！

别急嘛，你可以使用 `Webhooks` ，这是  `github` 提供的功能，它能确保远程库的配置文件更新后客户端中的配置信息也得到更新。

噢噢，这还差不多。我去查查怎么用。

慢着，听我说完，`Webhooks` 虽然能解决，但是你了解一下会发现它根本不适合用于生产环境，所以基本不会使用它的。

而一般我们会使用 `Bus` 消息总线 + `Spring Cloud Config` 进行配置的动态刷新。

## 九、Spring Cloud Bus（消息总线Bus）

> 用于将服务和服务实例与分布式消息系统链接在一起的事件总线。在集群中传播状态更改很有用（例如配置更改事件，**配置更新了，但是其他系统不知道是否更新**）。

**需求：** 动态设置一些应用参数，参数改变的情况下不需要重启应用部署，可以实时生效

你可以简单理解为 `Spring Cloud Bus` 的作用就是**管理和广播分布式系统中的消息**，也就是消息引擎系统中的广播模式。当然作为 **消息总线** 的 `Spring Cloud Bus` 可以做很多事而不仅仅是客户端的配置刷新功能。

### 项目集成消息总线Bus

默认支持: kafka、rabbitmq
扩展支持：rocketmq（官方不支持, 开源项目可以支持RocketMQ的bus总线）

**扩展支持rocketmq步骤：**

- 下载spring-cloud-stream-bus-rocketmq项目到本地,并且安装到本地仓库 https://github.com/zhipingzhang/spring-cloud-stream-bus-rocketmq  （maven:install）

- 在order-server和product-server项目中添加actuactor和spring-cloud-starter-bus-rocketmq依赖. 默认使用的Topic是springCloudBus 可以通过spring.cloud.bus.destination 来指定

- 在order-server.yml和product-server.yml配置文件中增加RocketMQ的配置
```yml
spring:
  application:
    name: config-server
  cloud:
    stream:
      rocket:
        binder:
          name-server: 192.168.48.102:9876
        bindings:
          springCloudBusInput:
            consumer:
              broadcasting: true
      bindings:
        springCloudBusInput:
          group: test-group
```
```yml
# 暴露全部的监控信息
management:
	endpoints:
		web:
			exposure:
				include: "*"
```

- 需要刷新配置的地方，在类上面增加注解@RefreshScope

- 访问验证 post方式: http://localhost:8090/actuator/bus-refresh (手动刷新，而真正是不是手动的，下面配置自动触发配置有说)

**依赖：**
```java
# actuator
	<dependency>
	    <groupId>org.springframework.boot</groupId>
	    <artifactId>spring-boot-starter-actuator</artifactId>
	</dependency>

# bus-rocketmq
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-rocketmq</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>
```

### 流程

当git文件更改的时候，通过pc端用post 向端口为8090的config-client发送请求/bus/refresh／；此时8090端口会发送一个消息，由消息总线向其他服务传递，从而使整个微服务集群都达到更新配置文件。

而拥有了 `Spring Cloud Bus` 之后，我们只需要创建一个简单的请求，并且加上 `@ResfreshScope` 注解就能进行配置的动态修改了，下面我画了几张图供你理解。

![QQ图片20211010162510.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211010162510-7bfcac67d2994b6b8ad18e8302a314ea.png)

或：

![QQ图片20211010132439.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211010132439-1bf1a8b3253946bcbea4cd588b1dee2f.png)

或：

![springcloudbuss213dsfsd.jpg](/upload/2021/10/springcloud-bus-s213dsfsd-d65301b67fe342ccbe7913895c06c550.jpg)

### 消息总线实现原理分析

**SpringBoot的监控管理？**

默认情况是关闭监控管理, 需要通过配置开启监控：

**1. 配置监控开启**
```yml
management:
	endpoints:
		web:
			exposure:
				include: "*"
```

**2. 访问监控页面**
```
http://localhost:9100/actuator
http://localhost:9100/actuator/env springboot环境配置
http://localhost:9100/actuator/beans  获取对应的Bean信息
```

**3. 可以通过post请求访问更新最新的环境配置, 并且会清除缓存中的Bean对象(使用@RefreshScope标记的类)**
```
 http://localhost:8090/actuator/refresh
```

**4. 可以通过在类上标记注解@RefreshScope 来声明该Bean对象是一个延迟加载的实例Bean对象**

**5. 在bus消息总线环境可以通过**
 `http://localhost:8090/actuator/bus-refresh` 进行消息总线的通知更新。

**配置自动触发配置？**

我们可以在中央仓库的配置更新以后, 通过事件机制webhook通知到对应的配置中心, 然后配置中心可以将请求转发到/actuator/bus-refresh, 完成消息总线的通知

**1. 导入依赖**
```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-monitor</artifactId>
</dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-rocketmq</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

**2. 配置属性**
```yml
spring:
  application:
    name: config-server
  cloud:
    stream:
      rocket:
        binder:
          name-server: 192.168.48.102:9876
        bindings:
          springCloudBusInput:
            consumer:
              broadcasting: true
      bindings:
        springCloudBusInput:
          group: test-group
```

**3. 调用回调方法 使用POST请求（9100为网关）**
```
http://127.0.0.1:9100/monitor?path=*
```

> **生产者消费者之间的调用?**

> 1 通过http发送：同步调用  依赖：RestTemplate、HttpClient等等

> 2 通过RPC调用（dubbo）：同步调用  Dubbo RPC

> 3 使用异步方式： 消息中间件：RocketMQ、Kafka、RabbitMQ、ActiveMQ等等

## 十、总结

这篇文章中我带大家初步了解了 `Spring Cloud` 的各个组件，他们有

* `Eureka` 服务发现框架
* `Ribbon` 进程内负载均衡器
* `Open Feign` 服务调用映射
* `Hystrix` 服务降级熔断器
* `Zuul` 微服务网关
* `Config` 微服务统一配置中心
* `Bus` 消息总线

如果你能这个时候能看懂文首那张图，也就说明了你已经对 `Spring Cloud` 微服务有了一定的架构认识。