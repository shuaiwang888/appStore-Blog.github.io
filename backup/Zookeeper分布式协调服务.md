## 一、分布式的基本概念

### 1.1 系统高可用
所谓的系统高可用，主要是指两个方面：
- 系统的健壮性, 不允许系统出现单点故障；
- 系统的处理能力, 可以提高系统的处理能力, 保证系统的运行效率

**集群**
 所谓的集群, 主要是原来使用的是一台服务器处理, 现在使用多台服务器保障系统的运行，主要分为：

- **主备集群:** 主要有一个主要节点提供服务, 另外的节点主要是出于备份状态, 平时不提供工作, 一旦主节点出现问题, 备份节点启动运行, 提供正常的服务；

![QQ图片20211005131557.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211005131557-63e5f8b0303f4921a8ddc066f5491de4.png)

- **主从集群:** 集群中的节点都提供服务, 但是每台服务器的角色可能不一样, 比如配置数据库的读写分离, 主数据库可能是写操作, 对于实时性要求不高的读操作就使用从数据库，该模式对于主从有数据传输延迟性；

![QQ图片20211005131606.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211005131606-2462413ea43e4b3195c95989cd2e1f96.png)

- **普通集群:** 集群中的节点提供的功能是一样的, 所有的节点没有主从之分,主要是提高系统的高可用；

![QQ图片20211005131608.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211005131608-fd7c58a463614a1dbb26b212de33740c.png)

**分布式**

分布式是系统部署方式：
- 比如我们的业务系统, 部署一个业务系统需要的环境(应用服务(Tomcat)+数据库服务(MySQL))；

- 如果我们把Tomcat和数据库MySQL服务部署在同一台服务器, 我们称之为单机部署, 这样Tomcat和MySQL之间的网络开销可以忽略(直接走127.0.0.1不会消耗网络)；

- 如果我们把Tomcat和数据库MySQL服务部署在不同的服务器, 我们称之为分布式应用, 因为应用服务器和数据库服务器之间需要走网络通信, 当然, **我们把所有需要走网络这种部署方式称之为"分布式应用"**；

![QQ图片20211005132216.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211005132216-e8540e32c1da481899ca7203721bf30b.png)

**微服务**
当使用单体应用时：其中一个模块挂了，整个系统（crm）都会访问不了；如果只是针对整个jar包（crm.jar）部署的话，就没法针对高并发模块进行单独的增强扩容或者添加服务器了；对于系统维护/后期开发不方便，若想要单独操作一个模块，需要把整个系统停掉，导致其他模块不可用。

而微服务就可以解决上述的问题：通过模块化部署，可以避免一个模块宕机导致整个系统不可用问题；可以单独针对高并发场景（秒杀，订单）进行扩容或者添加设备；方便开发人员独立开发，互不干扰。

微服务一定是分布式, 但是分布式不一定是微服务：
- 所谓"微服务" 指的是我们系统的一个架构设计方式；

- 以前我们把所有的功能都放到一个项目(应用中), 这种方式我们称之为单体应用；

- 随着项目开发的功能变多, 架构变强, 我们需要根据模块来进行划分, 每个模块之间通过服务之间的网络调用, 我们称之为微服务架构，进行模块化开发；

   ![QQ图片20211005132415.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211005132415-701746040b644f709219e27d4cb4493f.png)

### 1.2 分布式的概念

分布式在大数据时代非常重要而且基础的一个概念, 我们在大数据时代, 面临的主要问题和解决办法有：
- 海量数据的存储  ---> 分布式存储，可动态扩容
- 海量数据的运算  ----> 分布式计算
- 高并发的请求     -----> 分布式系统

**分布式存储**

![QQ图片20211005132648.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211005132648-3311c60b20b840139507738c792cfaa7.png)

**分布式计算（分而治之）：模块化计算，然后汇总；** 强调的是把一个大的计算任务分发成多个小的计算任务, 并行运算, 最终把结果进行汇总；对于分布式计算, 我们强调的是移动运算, 而不是移动数据；

![QQ图片20211005132838.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211005132838-2794d4eb727649dc8b6424eabad515cd.png)

**分布式系统**

略

### 1.3 分布式协调服务

- 所谓分布式：主要是指我们的一个整个的应用是由部署在多个机器上的服务去统一完成， 对于部署在多个机器上的应用， 通常是有不同的角色，比如有Master和Slave角色的区分，对于Zookeeper集群， 也会有多个节点，主要包括leader节点和follower节点 ；

- 所谓协调服务: 主要是指我们的Zookeeper去在我们的分布式系统中充当一个协调者的角色，帮助我们的具体的业务系统之间的相互协调，保证系统的正常运行；

- 一般配置2n+1台主从节点；

**Zookeeper服务步骤：**

1. 启动分布式协调服务zookeeper；
2. 启动秒杀服务, 启动的时候往zookeeper注册自己的信息(ip地址+端口) 和名称；
3. 启动客户端, 去zookeeper中找到对应的秒杀服务的地址列表信息；
4. 从地址列表信息中随机选取一个节点地址和端口；
5. 客户端直接发送远程服务调用(RPC调用/HTTP调用)，这里是发送请求到对应的秒杀服务节点地址和端口，这个过程和Zookeeper是没有关系的（注意：客户端不仅仅是浏览器）；

![QQ图片20211005134448.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211005134448-72adff01bed84003baf9f3da77b937ad.png)


## 二、Zookeeper基础

### 2.1 应用场景

> **1. 服务器在线感知(数据发布/订阅)**

数据发布/订阅：通过Watcher机制可以很方便地实现数据的发布和订阅。当你将数据发布到Zookeeper被监听的节点上，其他机器可通过监听Zookeeper上的节点的变化来实现配置的动态更新；

**Zookeeper如何实现在线感知：**
- 所有服务器启动的时候，都向Zookeeper中的一个指定目录写入一个数据/server/serverxxx（注册信息）；
- 客户端连接Zookeeper，获取到/servers/目录下面所有的可用的服务信息；
- 客户端监听/servers这个目录下面的数据改变， 如果Zookeeper的这个目录下的数据发生改变，那么会及时的通知客户端数据已经发生改变（观察者模式）；
- 客户端收到Zookeeper的通知，及时的去获取最新的数据；
- 当应用程序宕机，在Zookeeper中会及时的删除对应的服务信息（心跳机制检查是否宕机：每个固定时间有秒杀服务发送请求到Zookeeper以表示连接未中断）；

> **2. 主备协调**

对于我们的集群环境中的多个机器， 其中一台是处于活跃状态，可以正常的提供对应的服务， 另外的服务器是出于备份状态， 只有当活跃状态的机器出现问题，不能提供服务的时候，才会把备份状态的机器切换为活跃状态；

**Zookeeper如何实现主备协调：**
- Server01是active状态的机器, 向Zookeeper中的/server/目录写入数据server01；
- Server02是stadyBy状态的机器, 向Zookeeper中监听/server/目录中的数据改变；
- 当server01出现宕机, Zookeeper会删除/server/server01这个数据,并且会及时的通知到server02；
- Server02收到Zookeeper的事件通知，启动服务并且向Zookeeper注册自己的信息/server/server02；
- 客户端只需要向Zookeeper中找到/server/目录下面的服务信息,既可以正常的访问服务；

  ![QQ图片20211005134812.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211005134812-f900419f5d9d47a1907f9f9afb301916.png)

> **3. 配置管理**

在我们的大型的应用中，对于一个系统的配置会有很多需要配置的参数，比如说数据库的配置，Tomcat的线程数的配置等等, 如果我们不使用统一的配置管理中心的话, 需要在每个应用服务去进行一个单独的配置，这样操作比较麻烦，而且还容易出错，我们可以适用一个统一的配置管理；

**Zookeeper如何实现配置管理：**
- 提供一个配置管理程序, 用于向Zookeeper中写入对应的数据, 主要包括属性名称和属性值；
- 所有的服务启动的时候都去读取zookeeper中的配置信息,加载到应用程序,完成系统的正常启动；

  ![QQ图片20211005135050.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211005135050-fc18322254ad401687f1450998a2ac71.png)

> **4. 命名服务**

命名服务就是指通过指定的名字来获取资源或者服务的地址。Zookeeper会在自己的文件系统上（树结构的文件系统）创建一个以路径为名称的节点，它可以指向提供的服务的地址，远程对象等。简单来说使用Zookeeper做命名服务就是用路径作为名字，路径上的数据就是其名字指向的实体，**所以之后客服端就用名称直接发送RPC请求到对应服务器的对应端口**。

![QQ图片20211005135212.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211005135212-9df350fb4e124b6397a60f57fefc578a.png)

> **5. 分布式锁**

通过创建唯一节点获得分布式锁，当获得锁等等一方执行完相关代码或者挂掉之后就释放锁；在分布式系统的架构设计中,有时候需要保证分布式系统中的对于某些接口的原子性的操作, 我们需要控制在同一时刻只能有一个应用程序可以正常操作,其他的程序必须等待在操作的程序完成以后才可以正常的操作数据：

**Zookeeper如何实现分布式锁：**	
- 所有需要访问生成ID的接口的服务的应用都去Zookeeper的指定目录生成一个自己的数据/lock/serverxxx；
- 所有的业务系统都判断一下自己生成的服务地址是否是所有的地址列表中的最小的一个, 如果是最小的一个,则可以正常访问接口,否则,出于等待状态；
- 当获得锁对象的应用访问完生成ID的接口以后, 我们需要删除自己在Zookeeper中的服务列表,然后在进行对应的操作；
- 其他的应用再去判断自己的数据是否是Zookeeper中的地址列表中的最小的一个数据,如果是的话,即可以获取到锁对象,开始资源对象的访问；

  ![QQ图片20211005135444.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211005135444-2a1f929ca555424faa7975b935ab2a94.png)

### 2.2 Zookeeper的安装

> **1. 安装规划**

- 集群安装的时候建议使用2n+1；
- 对于集群中的节点, 我们有leader节点和follower节点, 当leader节点挂掉之后, 我们需要在集群中选择一个节点作为leader节点, 其中选举的策略是少数服从多数；
- 也可以使用Docker安装Zookeeper，如 `docker pull zookeeper:3.5.8`

  ![QQ图片20211005140142.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211005140142-9d37400f912240b98e3449cc778bd521.png)

> **2. 安装JDK(略)**

> **3. 安装Zookeeper**

下载并上传安装包到服务器：
```
https://downloads.apache.org/zookeeper/zookeeper-3.6.0/apache-zookeeper-3.6.0-bin.tar.gz
```

解压到指定目录：
```
tar -zxvf apache-zookeeper-3.6.0-bin.tar.gz  -C /usr/local/
```

修改配置文件（配置文件所在的目录：zookeeper/conf）：
```
cp zoo_sample.cfg zoo.cfg   #拷贝配置文件

#在配置文件中添加如下内容
dataDir=/root/apps/zookeeper/data  // 文件数据存放位置
server.1=lab100:2888:3888   // 分布式的各个服务器的端口地址
server.2=lab101:2888:3888
server.3=lab102:2888:3888  
```

创建数据存储目录：
```
#在/root/apps/zookeeper创建data目录
mkdir -p /root/apps/zookeeper/data
```

创建一个标志文件myid
```
cd /root/apps/zookeeper/data
echo 1 > myid    // 重定向输出到myid文件中
```

把安装好的配置拷贝到其他的机器：
```
注意: 一定要修改myid文件
到lab101上, 修改myid 为 2
到lab102上, 修改myid 为 3
```

配置Zookeeper的环境变量：
```
vi /etc/profile  #编辑修改文件

#文件内容
export ZOOKEEPER_HOME=/root/apps/zookeeper
export PATH=$PATH:$ZOOKEEPER_HOME/bin
```

启动服务：
```
#服务器jar下的bin/zkServer.sh，即zookeeper-service-01/bin/zkServer.sh

zkServer.sh start # 启动命令
zkServer.sh status # 查看状态命令
zkServer.sh stop # 停止服务命令
zkServer.sh restart# 停止服务命令
```

### 2.3 Zookeeper的核心配置

配置文件（zookeeper/conf/zoo.cfg）中的信息：
```
tickTime=2000   # 心跳的时间单位：毫秒值
initLimit=10    # 启动集群的初始化的事件的时间限制（10个心跳单位=20s）
syncLimit=5     # 集群服务器同步数据的事件5个心跳单位, 10s钟
dataDir=/root/apps/zookeeper-3.4.5/data    #数据的快照存储目录
clientPort=2181   #服务端提供客户端访问的连接接口

#集群中的服务列表1 2 3代表的是一个服务id，需要和myid文件中的内容一致，而且在1-154之间；
server.1=lab100:2888:3888  // 2888：数据同步端口；3888：选举端口
server.2=lab101:2888:3888
server.3=lab102:2888:3888  
```

### 2.4 Zookeeper的核心工作机制

ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现.雅虎的一个开源产品.，Apache的顶级开源项目；

它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。

> **①zookeeper特性**

- Zookeeper是由一个leader，多个follower组成的集群(3个节点最佳实践)；
- **为什么一般是2n+1个节点呢？**  因为Leader节点的选举机制是少数服从多数，3个节点的话2个以上同意就行，如果出现了宕机，一台宕机仍可以访问，两台才不行；而如果选用2n个节点（4个），就必须3台以上选举才有效，宕机两台就不能访问，所以效果跟3个节点一样！而且集群节点越多，就比较影响写数据的性能，因为写数据只能在Leader节点，并且需要同步到其他节点才能访问。
- **全局数据一致：** 每个server节点保存一份相同的数据副本，client无论连接到哪个server，数据都是一致的； 
- **分布式读写：** 更新请求转发，由leader实施，更新请求顺序进行，来自同一个client的更新请求按其发送顺序依次执行；
- **数据更新原子性：** 一次数据更新要么成功（半数以上节点成功），要么失败；
- **实时性：** 在一定时间范围内，client能读到最新数据(毫秒级别) 1M；
- **可靠性：** 一旦一次更改请求被应用，更改结果就会被持久化，直到下一次更改覆盖；

> **②zookeeper数据结构**

- 层次化的目录结构，命名符合常规文件系统规范(见下图)；
- 每个节点在zookeeper中叫做znode,并且其有一个唯一的路径标识；
- 节点Znode可以包含数据(只能存储很小量的数据，<1M)和子节点（但是EPHEMERAL类型（临时节点）的节点不能有子节点，下一页详细讲解）；
- 客户端应用可以在节点上设置监视器（后续详细讲解）；

> **③节点类型**

Znode有四种形式的目录节点：
```
PERSISTENT  		   #持久化节点
PERSISTENT_SEQUENTIAL      #持久化顺序节点
EPHEMERAL		   #临时节点
EPHEMERAL_SEQUENTIAL       #临时节点顺序节点
```

创建znode时设置顺序标识，znode名称后会附加一个值，顺序号是一个单调递增的计数器，由父节点维护；

在分布式系统中，顺序号可以被用于为所有的事件进行全局排序，这样客户端可以通过顺序号推断事件的顺序；

短暂（ephemeral）：断开连接自己删除；持久（persistent）：断开连接不删除；

### 2.5 Zookeeper基本操作命令

> **①服务器的启动和监控**

```
zkServer.sh  start | stop | restart | status
jps   // 查看java进程  QuorumPeerMain:这是zookeeper的进程；
netstat -natl   // 查看进程号
netstat -ntpl
```

> **②客户端连接**

```
zkCli.sh -server localhost:2181  // 默认就是本机的2181
// 如果是连接当前本机地址, 可以使用 zkCli.sh
// help: 查看命令, 可以查看具体可以执行的命令，主要是增删改查的基本操作,跟数据库一样.
```

> **③创建节点**

```
create [-s] [-e] [-c] [-t ttl] path [data] [acl]

-s: 顺序节点 默认为否
-e: 临时节点 默认为否
-c: 创建一个容器节点
-t ttl: 创建一个持久化节点或者一个持久化顺序节点的时候, 指定存活时间
path: 完整的绝对路径|目录
data: 设置路径下存放的数据

// 创建持久化节点
create /pNod pNode

// 创建临时节点 (临时节点在会话关闭以后, 会自动删除，临时节点不允许有子节点)
create -e /eNode eNode

// 创建顺序节点 (对于顺序节点, 是允许进行节点重复创建, 默认会在节点后面添加一个全局唯一的顺序ID)
create /lock
create -s  /lock/getId
create -e -s  /lock/getId2

// 创建容器节点 (可以用来一个节点, 下面存放多个子节点, 当子节点删除完以后, 容器节点会自动删除)
create -c /cnode cnode
create -c /cnode/child01 child01
create -c /cnode/child02 child02
delete /cnode/child01
delete /cnode/child02
#默认配置：会通过定时器, 在一分钟没有子节点, 直接删除改节点
znode.container.checkIntervalMs

// 创建ttl节点 (可以为持久化和持久化顺序节点设置一个有效时间 默认时间是毫秒，默认情况下这个功能是禁用的, 需要在zkServer.sh中设置启动的属性进行修改)
#找到对应的start启动选项的启动参数, 添加一个：
-D -Dzookeeper.extendedTypesEnabled=true -Dzookeeper.emulate353TTLNodes=true
#创建一个有效时间的节点：
create -t 3000 /tNode tNode
```

> **④查看节点**

```
// 查看节点信息
ls [-s] [-w] [-R] path

[-s] : 显示统计信息
[-w]: 查看事件信息
[-R]: 显示递归目录 

// 获取节点数据
get [-s] [-w] path
-s: 显示统计信息
-w: 获取事件信息

# 举例
get -s /pNode
pNode # 获取到的节点数据
cZxid = 0x600000010  # 创建节点的事务Id
ctime = Tue Apr 21 16:23:31 CST 2020   #创建时间
mZxid = 0x600000010 # 修改的事务ID
mtime = Tue Apr 21 16:23:31 CST 2020 #修改时间
pZxid = 0x600000010 
cversion = 0 # 当前版本信息
dataVersion = 0 #数据版本信息
aclVersion = 0 #权限版本信息
ephemeralOwner = 0x0  #临时节点的会话ID
dataLength = 5 # 数据长度
numChildren = 0 # 子节点个数
```

> **⑤修改节点数据**

```
set [-s] [-v version] path data

-s 设置过程显示节点的状态信息
-v: 使用CAS设置数据, 使用乐观锁, 可以使用stat从dataVersion中找到版本
set -v 0 /node2 node3
set  /node2 node3
```

> **⑥删除节点**

```
// 单个节点删除
delete [-v version] path

-v: 在并发的时候, 使用乐观锁进行删除，version是节点的版本
delete /node2

// 删除所有节点(递归删除)
deleteall /pNod
```

> **⑦绑定事件**

```
// 绑定一次事件

// 绑定永久的事件
```

## 三、Zookeeper应用

### 3.1 Java客户端操作

> **①自带的zkclient**

```
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.6.0</version>
</dependency>
```

> **②Apache开源的Curator**

Curator是一套Zookeeper Java开源框架，相比比喻自带的客服端Zookeeper来说，它的封装更完善，各种API都可以比较方便的使用。
```
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>4.3.0</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-client</artifactId>
    <version>4.3.0</version>
</dependency>
```
**连接Zookeeper客户端：** 通过 CuratorFrameworkFactory 创建 CuratorFramework 对象，然后再调用 CuratorFramework 对象的 start() 方法即可！

> **③Apache开源的ZkClient(com.101tec)**

```
<dependency>
    <groupId>com.101tec</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.11</version>
</dependency>
```
版本在18年就停止更新,用前面两个就可以了!

### 3.2 Java客户端API

> **①创建会话**

创建会话是在SpringBoot的入口程序下：
```
@SpringBootApplication
@ConfigurationProperties(prefix = "zookeeper.server")
@Data
public class ZookeeperApiDemo02Application {
    private String SERVER_ADDR;
    private int SESSION_TIMEOUT;

    public static void main(String[] args) {
        SpringApplication.run(ZookeeperApiDemo02Application.class, args);
    }

    @Bean
    public ZooKeeper zooKeeper() throws Exception{
        System.out.println("SERVER_ADDR = " + SERVER_ADDR);
        System.out.println("SESSION_TIMEOUT = " + SESSION_TIMEOUT);
        ZooKeeper client = new ZooKeeper(SERVER_ADDR, SESSION_TIMEOUT, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                System.out.println("event = " + event);
                if (event.getState() == Event.KeeperState.SyncConnected) { 
                    System.out.println("zookeeper 连接成功...");
                }
            }
        });
        return client;
    }
}

// Event.KeeperState.SyncConnected  会话的状态:连接状态
```

> **②创建节点**

创建节点以及下面的API是在controller控制器中: 
```
    @RequestMapping("createNode")
    public String createNode(String path, String data, String type) throws Exception{
	
	@Autowired
	private Zokeeper zookeeper;

        String result = zooKeeper.create(path, data.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.valueOf(type));
        return result;
    }

// ZooDefs.Ids.OPEN_ACL_UNSAFE 开放所有的权限
```

> **③获取节点中的数据**

**同步获取：** 是指必须等到data返回之后，才执行下面的代码。
```
@RequestMapping("getData")
public String getData(String path) throws Exception{
    byte[] data = zooKeeper.getData(path, new Watcher() {
        @Override
        public void process(WatchedEvent event) {
            System.out.println("hhhh");
            System.out.println("event = " + event);
        }
    }, new Stat());
    return new String(data);
};
```

**异步回调：** 
```
    /**
     * 异步数据处理：不会直接返回数据，只是在后面进行callback进行回调,用处不大，因为本来节点存储数据<1M
     * @param path
     * @return
     * @throws Exception
     */
    @RequestMapping("getDataAsync")
    public String getDataAync(String path) throws Exception{
        zooKeeper.getData(path, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                System.out.println("hhhh");
                System.out.println("event = " + event);
            }
        }, new AsyncCallback.DataCallback() {
            @Override
            public void processResult(int rc, String path, Object ctx, byte[] data, Stat stat) {
                System.out.println("data = " + new String(data));
                System.out.println("stat = " + stat);
                System.out.println("ctx = " + ctx);
            }
        }, "测试数据");
        return " 正在获取数据中";
    };
```

**获取子节点列表：** 
```
    /**
     * 获取子节点列表，当然可以没有绑定watch
     */
    @RequestMapping("getChildren")
    public  List<String> getChildren(String path) throws Exception{
        List<String> children = zooKeeper.getChildren(path, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                System.out.println("ZookeeperController.process");
                System.out.println("event = " + event);
                System.out.println(event.getState());
            }
        });
        return children;
    }
```

> **④判断节点是否存在**

```
@RequestMapping("exists")
public  boolean exists(String path) throws Exception{
    Stat stat = zooKeeper.exists(path, false);
    return stat!=null ;
}
```

> **⑤删除节点：** 删除和更新操作都会需要版本信息，这是因为防止并发而用到的乐观锁，需要在操作之前就去看看节点是否存在；

```
// 删除节点
@RequestMapping("delete")
public  boolean delete(String path) throws Exception{
    Stat stat = zooKeeper.exists(path, false); // 先判断是否存在
    if(stat!=null){
        zooKeeper.delete(path,stat.getVersion()); // 加上版本信息
    }
    return true;
}

// 更新数据
@RequestMapping("update")
public  boolean update(String path,String data) throws Exception{
    Stat stat = zooKeeper.exists(path, false);
    if(stat!=null){
        zooKeeper.setData(path,data.getBytes(),stat.getVersion());
    }
    return true;
}
```

> **⑥事件处理：**

**绑定一次事件：**
```
    @RequestMapping("addWatch2")
    public List<String> addWatch2(String path) throws Exception{
        //1 先获取所有的子节点
	//2 绑定一个监听事件
        Watcher watcher = new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                if (event.getType() == Event.EventType.NodeChildrenChanged) {
                    try {
                        List<String> children = zooKeeper.getChildren(path, this);
                        System.out.println("children = " + children);
                    } catch (KeeperException e) {
                        e.printStackTrace();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        };
        List<String> children = zooKeeper.getChildren(path, watcher);
        return children;
    }
```

**绑定永久事件**
```
    @RequestMapping("addWatch")
    public List<String> addWatch(String path) throws Exception{
        //1 先获取所有的子节点
        List<String> children = zooKeeper.getChildren(path, false);
        //2 绑定一个监听事件
        zooKeeper.addWatch(path, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                if(event.getType()== Event.EventType.NodeChildrenChanged){
                    System.out.println("子节点数据发送改变");
                    System.out.println("重新获取子节点数据");
                    try {
                        List<String> children1 = zooKeeper.getChildren(path, false);
                        System.out.println("children1 = " + children1);
                        System.out.println("=========================");
                    } catch (KeeperException e) {
                        e.printStackTrace();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }else if(event.getType()== Event.EventType.NodeDataChanged){
                    System.out.println("节点数据发生改变");
                    try {
                        byte[] data = zooKeeper.getData(path, false, new Stat());
                        System.out.println("获取到的数据是:"+new String(data));
                    } catch (KeeperException e) {
                        e.printStackTrace();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }, AddWatchMode.PERSISTENT);
        return children;
    }
```

**递归绑定事件：** 对于创建的节点以及子节点都绑定事件
```
    @RequestMapping("addWatch")
    public List<String> addWatch(String path) throws Exception{
        //1 先获取所有的子节点
        List<String> children = zooKeeper.getChildren(path, false);
        //2 绑定一个监听事件
        zooKeeper.addWatch(path, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                if(event.getType()== Event.EventType.NodeChildrenChanged){
                    System.out.println("子节点数据发送改变");
                    System.out.println("重新获取子节点数据");
                    try {
                        List<String> children1 = zooKeeper.getChildren(path, false);
                        System.out.println("children1 = " + children1);
                        System.out.println("=========================");
                    } catch (KeeperException e) {
                        e.printStackTrace();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }else if(event.getType()== Event.EventType.NodeDataChanged){
                    System.out.println("节点数据发生改变");
                    try {
                        byte[] data = zooKeeper.getData(path, false, new Stat());
                        System.out.println("获取到的数据是:"+new String(data));
                    } catch (KeeperException e) {
                        e.printStackTrace();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }, AddWatchMode.PERSISTENT_RECURSIVE);
        return children;
    }
```

### 3.3 Zookeeper应用实战

> **服务器的动态感知**

**1. 业务分析**

![QQ图片20211006101533.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211006101533-21ba8b48c35a48abb1b2739da9a73566.png)

**2. 服务注册**
```
一、创建一个SpringBoot项目
# 1 导入对应的依赖包
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.6.0</version>
</dependency>

# 2 在SpringBoot程序入口实例化一个Zookeeper的客户端连接，连接成功之后就注册对应信息

# 3 在连接成功以后,开始创建一个对应的临时顺序节点, 注册自己的ip和端口

二、启动项目，连接zk，并且注册地址和端口信息

三、具体实现
# 1 配置文件，下面是yml文件格式的配置文件zookeeper.server
zookeeper:
  server:
    SERVER_ADDR: 192.168.48.100:2181
    SESSION_TIMEOUT: 60000
    PATH: servers
    SUB_DIR: server
server:
  port: 9898
  host: 127.0.0.1

# 2 代码
@SpringBootApplication
@ConfigurationProperties(prefix = "zookeeper.server")
@Data
public class ServerRegistry02Application  {

    private String SERVER_ADDR;
    private int SESSION_TIMEOUT;
    private String PATH;
    private String SUB_DIR;
    @Value("${server.host}")
    private String host;
    @Value("${server.port}")
    private String port;
    private ZooKeeper zooKeeper;

    public static void main(String[] args) {
        SpringApplication.run(ServerRegistry02Application.class, args);
    }

    @Bean
    public ZooKeeper zooKeeper() throws  Exception {
          zooKeeper = new ZooKeeper(SERVER_ADDR, SESSION_TIMEOUT, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                System.out.println(event);
                if(event.getState() == Event.KeeperState.SyncConnected){
                    System.out.println("event = " + event);
                    System.out.println("服务器连接成功");
                    //开始注册自己的信息
                    String addr=host+":"+port;
                    try {
                        zooKeeper.create("/"+PATH+"/"+SUB_DIR,addr.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
                    } catch (KeeperException e) {
                        e.printStackTrace();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        });
        return zooKeeper;
    }
}
```

**3. 服务发现**
```
一、创建一个SpringBoot项目
# 1 导入依赖
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.6.0</version>
</dependency>

# 2 创建一个zookeeper客户端连接

# 3 连接成功以后,获取地址列表

# 4 绑定子节点改变事件

二、启动项目，连接zk，并且获取服务地址列表

三、注册永久的事件监听

四、具体实现
# 1 配置文件
zookeeper:
  server:
    SERVER_ADDR: 192.168.48.100:2181
    SESSION_TIMEOUT: 60000
    PATH: servers

# 2 代码
@SpringBootApplication
@ConfigurationProperties(prefix = "zookeeper.server")
@Data
public class ServerDiscover02Application {

    private ZooKeeper zooKeeper=null;
    private String SERVER_ADDR;
    private int SESSION_TIMEOUT;
    private String PATH;
    public static  volatile List<String> serverAddr=new ArrayList<>();

    public static void main(String[] args) {
        SpringApplication.run(ServerDiscover02Application.class, args);
    }

    @Bean
    public ZooKeeper zooKeeper() throws  Exception {
        zooKeeper = new ZooKeeper(SERVER_ADDR, SESSION_TIMEOUT, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                System.out.println(event);
                if(event.getState() == Event.KeeperState.SyncConnected){
                    System.out.println("event = " + event);
                    System.out.println("服务器连接成功");
                    try {
                        //1 获取对应的服务器列表
                        serverAddr = zooKeeper.getChildren("/"+PATH, false);
                        System.out.println("最新服务器列表信息:" + serverAddr);
                        //2 如果有修改,及时更新, 注册子节点事件简体
                        zooKeeper.addWatch("/"+PATH, e->{
                            if(e.getType() == Event.EventType.NodeChildrenChanged){//子节点改变事件
                                try {
                                    serverAddr = zooKeeper.getChildren("/"+PATH, false);
                                    System.out.println("最新服务器列表信息:" + serverAddr);
                                } catch (Exception e1) {
                                    e1.printStackTrace();
                                }
                            }
                        }, AddWatchMode.PERSISTENT);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }
        });
        return zooKeeper;
    }
}
```

> **分布式锁业务处理**

**1. 为什么程序中需要锁**

- **多任务环境：** 多个任务同时执行, 可以是多线程, 也可以是多进程；

- **多个任务的资源共享操作：** 所有的任务都需要对同一资源进行写操作；

- **对资源的访问是互斥的：** 对于资源的访问, 不同多个任务同时执行, 同一时间只能一个任务访问资源, 其他的任务处于阻塞状态；

**2. 锁的基本概念**

- **竞争锁：** 任务通过竞争获取锁才能对该资源进行操作，分为公平竞争（按照一定的顺序, 先来先执行）和非公平竞争（没有顺序, 不管先后顺序执行）
- **占有锁：** 当有一个任务在对资源进行更新时；
- **任务阻塞：** 其他任务都不可以对这个资源进行操作；
- **释放锁：** 直到该任务完成更新 释放资源；

**3. 锁的应用场景**

单机环境(在一个虚拟机中)：
```
// 业务实现
public class OrderIDGenerator {

    private int count=0;

    public synchronized String getId(){
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-dd-mm");
        String format = sdf.format(new Date());
        try {
            TimeUnit.MILLISECONDS.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return format+"-"+(++count);
    }
}

// 测试实现
public class OrderService implements Runnable {
    private OrderIDGenerator orderIDGenerator=null;
    private static CountDownLatch countDownLatch=new CountDownLatch(50);
    private static Set<String> result=new HashSet<>(50);

    public OrderService(OrderIDGenerator orderIDGenerator) {
        this.orderIDGenerator = orderIDGenerator;
    }

    public static void main(String[] args) throws Exception{
        OrderIDGenerator idGenerator = new OrderIDGenerator();
        System.out.println("开始模拟多线程生成订单号");
        for(int i=0;i<50;i++){
            new Thread(new OrderService(idGenerator)).start();
        }
        countDownLatch.await();
        System.out.println("生成的订单号个数:"+result.size());
        System.out.println("======================");
        for (String order : result) {
            System.out.println(order);
        }
    }

    @Override
    public void run() {
        result.add(orderIDGenerator.getId());
        countDownLatch.countDown();
    }
}
```

分布式环境_同名节点：

1. 业务分析
![QQ图片20211006103251.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211006103251-e253c5d55c1b41b89557f155f4f0c9e1.png)
2. 业务实现
```
@RestController
public class OrderController {

    private  RestTemplate restTemplate=new RestTemplate();

    @Autowired
    private ZooKeeper zooKeeper;

    private String path="/locks";
    private String node="/orderIdLock";

    @RequestMapping("createOrder")
    public String createOrder() throws Exception{
        //获取id
        if(tryLock()){
            //调用业务方法
            String id = restTemplate.getForObject("http://localhost:8080/getId", String.class);
            System.out.println("获取到的id:"+id);
            //释放锁
            unlock();
        }else {
            waitLock();
        }
        return "success";
    }
    //尝试获取id, 如果获取到了, 返回true, 否则返回false
    //竞争锁资源
    public boolean tryLock(){
        try {
            //因为不是顺序节点, 对于同一个路径, 只能创建一次
            String path = zooKeeper.create(this.path+this.node,
                    null, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
    //释放锁资源，就是删除指定节点(先判断是否存在)
    public void unlock(){
        try {
            Stat stat = zooKeeper.exists(this.path+this.node, false);
            zooKeeper.delete(this.path+this.node, stat.getVersion()); 
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    //阻塞状态
    public  void waitLock(){
        try {
	   // 绑定监听事件（子节点变化）,一次性事件
            zooKeeper.getChildren(this.path, new Watcher() {
                @Override
                public void process(WatchedEvent event) {
                    try {
                        createOrder();//重新创建订单
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            });
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

// 当没有获取到资源锁时，任务进入等待状态，表示写数据失败，
这时会绑定一个事件的通知（如/order/的子节点改变或者/order/lock节点的删除事件触发），
触发后再次尝试获取锁资源；重复以上操作，直到获取并执行了。

// 存在性能较差：
因为当同名节点删除之后，需要通知所有的等待的节点；
所有的等待节点都会去执行尝试获取锁；
如果没有获取到，又要重新绑定事件；
```

3. 分布式锁业务流程分析
![QQ图片20211006103629.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211006103629-4ba121b385b84d70a0637aa97190266d.png)

4. 分布式锁流程图
![QQ图片20211006103811.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211006103811-64dd593a9b694a908e393a97b57c9188.png)

**分布式环境_顺序节点：**

1. 业务实现
```
@RestController
public class Order02Controller {

    private RestTemplate restTemplate = new RestTemplate();

    @Autowired
    private ZooKeeper zooKeeper;

    private String path = "/locks02";
    private String node = "/orderIdLock";

    @RequestMapping("createOrder02")
    public String createOrder() throws Exception {
        String currentPath = zooKeeper.create(this.path + this.node,
                null, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
        currentPath=currentPath.substring(currentPath.lastIndexOf("/")+1);
        //获取id
        if (tryLock(currentPath)) {
            //调用业务方法
            String id = restTemplate.getForObject("http://localhost:8080/getId", String.class);
            System.out.println("获取到的id:" + id);
            //释放锁
            unlock(currentPath);
        } else {
            waitLock(currentPath);
        }
        return "success";
    }

    //尝试获取id, 如果获取到了, 返回true, 否则返回false
    //竞争锁资源
    public boolean tryLock(String currentPath) {
        try {
            //获取到所有的节点
            List<String> children = zooKeeper.getChildren(this.path, false);
            Collections.sort(children);
            if (StringUtils.pathEquals(currentPath, children.get(0))) {
                return true;
            } else {
                return false;
            }
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    //释放锁资源
    public void unlock(String currentPath) {
        try {
            Stat stat = zooKeeper.exists(this.path + "/" + currentPath, false);
            zooKeeper.delete(this.path + "/" + currentPath, stat.getVersion());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    //阻塞状态
    public void waitLock(String currentPath) {
        try {
            CountDownLatch count = new CountDownLatch(1);
            List<String> children = zooKeeper.getChildren(this.path, false);
            //获取到前一个节点
            Collections.sort(children);
            int index = children.indexOf(currentPath);
            if (index > 0) {
                String preNode = children.get(index - 1);
                //绑定前一个节点删除操作的事件
                zooKeeper.getData(this.path + "/" + preNode, new Watcher() {
                    @Override
                    public void process(WatchedEvent event) {
                        if (event.getType() == Event.EventType.NodeDeleted) {
                            try {
                                String id = restTemplate.getForObject("http://localhost:8080/getId", String.class);
                                System.out.println("获取到的id:" + id);
                                //释放锁
                                unlock(currentPath);
                                count.countDown();
                            } catch (Exception e) {
                                e.printStackTrace();
                            }
                        }
                    }
                }, new Stat());
            }
            count.await();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

2. 分布式锁流程
![QQ图片20211006104011.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211006104011-f0afe37984d14bf1bfb1931285609a85.png)

## 四、Zookeeper原理分析

### 4.1 Zookeeper的选举机制
我们在进行zk集群启动的时候，集群中会有Leader节点和Follower节点，一个集群中只会有一个Leader节点，并且在启动节点的时候Leader并不是固定的，而是通过一定的选举策略所产生；

在选择Leader节点的时候，需要进行投票(vote)，其中每个节点都可以进行投票，并且把自己的投票结果发送给其他的所有节点,其中投票的主要的信息vote 包含两个字段(myid,zxid): 其中myid代表的是服务器节点的id，zxid代表的是选举的全局事务ID；

**选举步骤：**
1. 当节点处于looking状态的时候, 会开始进行投票；

2. 第一次投票的时候, 永远是投自己的票vote(myid,zxid) 发送自己的ID信息；

3. 当收到其他节点发送的投票信息voteN(myidN,zxidN)时候, 会进行vote信息比较：①首先根据zxid进行比较, zxid值最大的准备选择为leader；②如果zxid相等, 那么根据myidN进行比较, 选择myIdN值   大的作为leader, 重新发出投票信息；

4. 当有超过半数的server选择相同的server作为leader，那么leader节点选择完成，便可以改变服务器的状态；

**注意：** 除了Leader和Follower两种角色之外，还有Observer角色：该角色为客服端提供读服务，如果是写服务则转发给Leader。不参与选举过程中的投票，也不参与"过半写成功"策略。只是在不影响写性能的情况下提升集群的读性能。此角色与Zookeeper3.3系列新增的角色；

**总结：**
- ZooKeeper 本身就是一个分布式程序（只要半数以上节点存活，ZooKeeper 就能正常服务）；
- 为了保证高可用，最好是以集群形态来部署 ZooKeeper，这样只要集群中大部分机器是可用的（能够容忍一定的机器故障），那么 ZooKeeper 本身仍然是可用的；
- ZooKeeper 将数据保存在内存中，这也就保证了高吞吐量和低延迟（但是内存限制了能够存储的容量不太大，此限制也是保持 znode 中存储的数据量较小的进一步原因）；
- ZooKeeper 是高性能的。 在“读”多于“写”的应用程序中尤其地明显，因为“写”会导致所有的服务器间同步状态。（“读”多于“写”是协调服务的典型场景）；
- ZooKeeper 有临时节点的概念。 当创建临时节点的客户端会话一直保持活动，瞬时节点就一直存在。而当会话终结时，瞬时节点被删除。持久节点是指一旦这个 znode 被创建了，除非主动进行 znode 的移除操作，否则这个 znode 将一直保存在 ZooKeeper 上；
- ZooKeeper 底层其实只提供了两个功能：① 管理（存储、读取）用户程序提交的数据；② 为用户程序提供数据节点监听服务；

![QQ图片20211006105858.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211006105858-9ee45686c9db443b92452d28ba780ab1.png)、

### 4.2 一致性协议分析

**1. Paxos算法**
Paxos算法是Lamport宗师提出的一种基于消息传递且具有高度容错性的分布式一致性算法，使其获得2013年图灵奖。
- 分为两个阶段提交；
- 少数服从多数；
- 避免单点故障；

![QQ图片20211006110840.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211006110840-aeebf475a38f4ac3b301de7ba2ab5518.png)

**2. CAP定理**
CAP定理: 对于共享数据系统，最多只能同时拥有CAP其中的两个，没法三者兼顾。任两者的组合都有其适用场景；不同类型的业务可以也应当区别对待；Zoopeeker的处理方式保证了CP（数据一致性）。

![QQ图片20211006110954.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211006110954-4e1dff7be4044e8486392d4f984db493.png)

**3. BASE理论**

- BA: Basic Availability 基本业务可用性（支持分区失败）
- S: Soft state 柔性状态（状态允许有短时间不同步，异步）
- E: Eventual consistency 最终一致性（最终数据是一致的，但不是实时一致）
- 原子性（A）与持久性（D）必须根本保障；
- 为了可用性、性能与降级服务的需要，只有降低一致性( C ) 与 隔离性( I ) 的要求；

**4. ZAB协议**
ZAB 协议是为分布式协调服务 ZooKeeper 专门设计的一种支持崩溃恢复的原子广播协议。在 ZooKeeper 中，主要依赖 ZAB 协议来实现分布式数据一致性；基于该协议，Zookeeper实现了一种主备模式的系统架构来保持集群中各副本之间的数据一致性；

**ZAB协议两种模式：消息广播和崩溃恢复**
- **崩溃恢复：** 当整个服务框架在启动过程中，或是当 Leader 服务器出现网络中断、崩溃退出与重启等异常情况时，ZAB 协议就会进入恢复模式并选举产生新的 Leader 服务器。当选举产生了新的 Leader 服务器，同时集群中已经有过半的机器与该 Leader 服务器完成了状态同步之后，ZAB 协议就会退出恢复模式。其中，所谓的状态同步是指数据同步，用来保证集群中存在过半的机器能够和 Leader 服务器的数据状态保持一致。

- **消息广播：** 当集群中已经有过半的 Follower 服务器完成了和 Leader 服务器的状态同步，那么整个服务框架就可以进入消息广播模式了。 当一台同样遵守 ZAB 协议的服务器启动后加入到集群中时，如果此时集群中已经存在一个 Leader 服务器在负责进行消息广播，那么新加入的服务器就会自觉地进入数据恢复模式：找到 Leader 所在的服务器，并与其进行数据同步，然后一起参与到消息广播流程中去。

 最典型的集群模式：Master/Slave模式(主备模式)。在这种模式下，通常Master服务器作为主服务器提供写服务，其他的Slave服务器(从服务器)通过异步复制的方式获取Master服务器最新的数据，从而提供读服务；

### 4.3 Zookeeper集群写数据流程

**写数据流程：**

![QQ图片20211006111359.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211006111359-8632d839d23a430c9f4cdfd0d035fe6b.png)

### 4.4 Zookeeper的其他事项

**1. 管理监控命令(运维)**
ZooKeeper 支持某些特定的四字命令字母与其的交互。用来获取 ZooKeeper 服务的当前状态及相关信息。可通过 telnet 或 nc 向 ZooKeeper 提交相应的命令 ：

```
echo stat|nc 127.0.0.1 2181  // 来查看哪个节点被选择作为follower或者leader 
echo ruok|nc 127.0.0.1 2181 // 测试是否启动了该Server，若回复imok表示已经启动;
echo dump| nc 127.0.0.1 2181  // 列出未经处理的会话和临时节点;
echo kill | nc 127.0.0.1 2181  // 关掉server;
echo conf | nc 127.0.0.1 2181  // 输出相关服务配置的详细信息;
echo cons | nc 127.0.0.1 2181  // 列出所有连接到服务器的客户端的完全的连接 / 会话的详细信息;
echo envi |nc 127.0.0.1 2181  // 输出关于服务环境的详细信息（区别于 conf 命令）;
echo reqs | nc 127.0.0.1 2181  // 列出未经处理的请求;
echo wchs | nc 127.0.0.1 2181  // 列出服务器 watch 的详细信息;
echo wchc | nc 127.0.0.1 2181  // 通过 session 列出服务器 watch 的详细信息，它的输出是一个与 watch 相关的会话的列表;
echo wchp | nc 127.0.0.1 2181  // 通过路径列出服务器 watch 的详细信息。它输出一个与 session 相关的路径;

// 安装nc
yum install -y nc

// 添加配置参数：编辑Zookeeper里的zoo.cfg配置文件为：
4lw.commands.whitelist=*
```

**2. 系统参数配置说明**
略