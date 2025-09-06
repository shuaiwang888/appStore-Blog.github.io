这篇文章主要是想通过一些问题，加深大家对于 Spring 的理解，所以不会涉及太多的代码！

下面的很多问题我自己在使用 Spring 的过程中也并没有注意，自己也是临时查阅了很多资料和书籍补上的。网上也有一些很多关于 Spring 常见问题/面试题整理的文章，我感觉大部分都是互相 copy，而且很多问题也不是很好，有些回答也存在问题。所以，自己花了一周的业余时间整理了一下，希望对大家有帮助。

## **什么是 Spring 框架?**

Spring 是一款开源的轻量级 Java 开发框架，旨在提高开发人员的开发效率以及系统的可维护性。

Spring 翻译过来就是春天的意思，可见其目标和使命就是为 Java 程序员带来春天啊！感动！

> 题外话 ： 语言的流行通常需要一个杀手级的应用，Spring 就是 Java 生态的一个杀手级的应用框架。

我们一般说 Spring 框架指的都是 `Spring Framework`，它是很多模块的集合，使用这些模块可以很方便地协助我们进行开发：比如说 Spring 自带 **IoC（Inverse of Control:控制反转）** 和 **AOP(Aspect-Oriented Programming:面向切面编程)** 以及 **DI（Dependency Injection:依赖注入）**、可以很方便地对数据库进行访问、可以很方便地集成第三方组件（电子邮件，任务，调度，缓存等等）、对单元测试支持比较好、支持 RESTful Java 应用程序的开发。

![](https://img-blog.csdnimg.cn/38ef122122de4375abcd27c3de8f60b4.png)

Spring 最核心的思想就是不重新造轮子，开箱即用！

Spring 提供的核心功能主要是 `IoC` 和 `AOP` 以及 `DI`。学习 Spring ，一定要把 IoC 和 AOP 的核心思想搞懂！

- Spring 官网：<https://spring.io/>
- Github 地址： https://github.com/spring-projects/spring-framework

## **列举一些重要的 Spring 模块？**

下图对应的是 Spring4.x 版本。目前最新的 5.x 版本中 Web 模块的 Portlet 组件已经被废弃掉，同时增加了用于异步响应式处理的 WebFlux 组件。

![Spring主要模块](https://images.xiaozhuanlan.com/photo/2019/e0c60b4606711fc4a0b6faf03230247a.png)

**Spring Core**

核心模块， Spring 其他所有的功能基本都需要依赖于该类库，主要提供 **IoC** 和 **DI** 功能的支持。

**Spring Aspects**

该模块为与 AspectJ 的集成提供支持。

**Spring AOP**

提供了面向切面的编程实现。

**Spring Data Access/Integration ：**

Spring Data Access/Integration 由 5 个模块组成：

- spring-jdbc : 提供了对数据库访问的抽象 JDBC。不同的数据库都有自己独立的 API 用于操作数据库，而 Java 程序只需要和 JDBC API 交互，这样就屏蔽了数据库的影响。
- spring-tx : 提供对事务的支持。
- spring-orm : 提供对 Hibernate 等 ORM 框架的支持。
- spring-oxm ： 提供对 Castor 等 OXM 框架的支持。
- spring-jms : Java 消息服务。

**Spring Web**

Spring Web 由 4 个模块组成：

- spring-web ：对 Web 功能的实现提供一些最基础的支持。
- spring-webmvc ： 提供对 Spring MVC 的实现。
- spring-websocket ： 提供了对 WebSocket 的支持，WebSocket 可以让客户端和服务端进行双向通信。
- spring-webflux ：提供对 WebFlux 的支持。WebFlux 是 Spring Framework 5.0 中引入的新的响应式框架。与 Spring MVC 不同，它不需要 Servlet API，是完全异步.

**Spring Test**

Spring 团队提倡测试驱动开发（TDD）。有了控制反转 (IoC)的帮助，单元测试和集成测试变得更简单。

Spring 的测试模块对 JUnit（单元测试框架）、TestNG（类似 JUnit）、Mockito（主要用来 Mock 对象）、PowerMock（解决 Mockito 的问题比如无法模拟 final, static， private 方法）等等常用的测试框架支持的都比较好。

## **Spring IOC**

### **谈谈自己对于 Spring IoC 的了解**

**IoC（Inverse of Control:控制反转）** 是一种设计思想，而不是一个具体的技术实现。IoC 的思想就是将原本在程序中手动创建对象的控制权，交由 Spring 框架来管理。不过， IoC 并非 Spirng 特有，在其他语言中也有应用。

**为什么叫控制反转？**

- **控制** ：指的是对象创建（实例化、管理）的权力
- **反转** ：控制权交给外部环境（Spring 框架、IoC 容器）

![](https://camo.githubusercontent.com/dd18c750a56f6d68652a5cb0e050e354ff20cbab0e8328f491a46e0b59323931/68747470733a2f2f67756964652d626c6f672d696d616765732e6f73732d636e2d7368656e7a68656e2e616c6979756e63732e636f6d2f6a6176612d67756964652d626c6f672f6672632d33363566616365623536393766303466333133393939333763303539633136322e706e67)

将对象之间的相互依赖关系交给 IoC 容器来管理，并由 IoC 容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。 IoC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。

在实际项目中一个 Service 类可能依赖了很多其他的类，假如我们需要实例化这个 Service，你可能要每次都要搞清这个 Service 所有底层类的构造函数，这可能会把人逼疯。如果利用 IoC 的话，你只需要配置好，然后在需要的地方引用就行了，这大大增加了项目的可维护性且降低了开发难度。

在 Spring 中， IoC 容器是 Spring 用来实现 IoC 的载体， IoC 容器实际上就是个 Map（key，value），Map 中存放的是各种对象。

Spring 时代我们一般通过 XML 文件来配置 Bean，后来开发人员觉得 XML 文件来配置不太好，于是 SpringBoot 注解配置就慢慢开始流行起来。

相关阅读：

- [IoC 源码阅读](https://javadoop.com/post/spring-ioc)
- [面试被问了几百遍的 IoC 和 AOP ，还在傻傻搞不清楚？](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486938&idx=1&sn=c99ef0233f39a5ffc1b98c81e02dfcd4&chksm=cea24211f9d5cb07fa901183ba4d96187820713a72387788408040822ffb2ed575d28e953ce7&token=1736772241&lang=zh_CN#rd)

**IoC容器的基本使用？**
- 之前的处理请求方式：请求来到对应的servlet方法，调用上层service的对应方法，再调用dao层的对应方法，最后在dao的具体实现类中处理，如此十分具有依赖和耦合性；

- 如何降低耦合？  工厂设计模式：

①配置工厂加载文件factory.properties；
```xml
personService=com.ws.service.impl.PersonServiceImpl
personDao=com.ws.dao.impl.PersonDaoImpl
```

②设计工厂模式：先加载配置文件，在根据传入的类名，实例化一个对象（service或dao）；
```java
public class GeneralFactory {
    private static Properties properties;
    
    // 加载配置工厂文件factory.properties
    static {
        try (InputStream is = GeneralFactory.class.getClassLoader().getResourceAsStream("factory.properties")) {
            properties = new Properties();
            properties.load(is);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    // 根据properties和传入的类名，实例化一个对象
    public static <T> T get(String name) {
        try {
            // 类名
            String clsName = properties.getProperty(name);
            Class cls = Class.forName(clsName);
            // newInstance()实例化对象
            return (T) cls.newInstance();
        } catch (Exception e) {
            e.printStackTrace();;
        }
        return null;
    }

    // 这里能分别拿到service/dao的实例化对象
    public static PersonService getService() {
        return get("service");
    }

    public static PersonDao getDao() {
        return get("dao");
    }
}
```
③在servlet或service具体实现类中直接通过工厂模式获取对应的实例化对象；
```java
public class PersonServlet {
    private PersonService service;
    
    // 有构造方法才是属性，才会在xml配置文件中向上引用，实例化对象！
    public void setService(PersonService service) {
        this.service = service;
    }

    public void remove() {
        service.remove(1);
    }

    public static void main(String[] args) {
        // 读取配置文件
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        PersonServlet servlet = ctx.getBean("personServlet", PersonServlet.class);
        servlet.remove();
    }
```
- 所以，Spring起的作用就是这里的工厂而且更强大；
- 如何在spring框架中实现呢？
①新建一个xml文件（复杂情况，而简单配置只需properties即可），直接编写的层级引用关系，便可自动实现实例化，即servlet对应引用service，而service对应引用dao；
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="personDao" class="com.ws.dao.impl.PersonDaoImpl"/>

    <bean id="personService" class="com.ws.service.impl.PersonServiceImpl">
        <property name="dao" ref="personDao"/>
    </bean>

    <bean id="personServlet" class="com.ws.servlet.PersonServlet">
        <property name="service" ref="personService"/>
    </bean>
</beans>
```
②在对应实现类中声明即可（private xxxService service;），但必须设置其构造方法：
```java
public void setService(xxxService service) {                                                                           				 
    this.service = service;                                                                     }
```
**因为有构造方法才是属性，才会在xml配置文件中进行向上引用，实现自动实例化对象；**

## **Spring 依赖注入**

1. 由前面的知识：在xxxserviceImpl中需要dao的实例，所以通过其构造方法setDao()将xxxservice需要的xxxDao注入到里面去，此称为依赖注入；
2. 常见的注入内容有3种：①bean(自定义类型)：就是上面1总结的那种方式；②基本类型，String，BigDecimal在applicationContext.xml通过value注入；③集合类型（数组、Map、List、Set、Properties）；
3. 常见的注入方式有2种:基于setter(属性)和基于constructor(构造方法)；

**基于setter(属性)**
![QQ图片20210922102307.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210922170748-6d5f906c0fd44500aee2c600508ed7a5.png)
![QQ图片20210922102310.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210922102310-60107af4e21646a1b24d83f5736a4d98.png)
![QQ图片20210922102313.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210922102313-a63bf49822214ebbae50b7744d1b1ac3.png)
![QQ图片20210922102317.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210922102317-154a4b7f56554f339e3879fd6ef9b31b.png)
![QQ图片20210922102320.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210922102320-62e28dc471ff44ac943c6cc12bff1629.png)

**基于constructor(构造方法)**
![QQ图片20210922102514.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210922102514-c4536aaf65bf4a68b41dea4749a86451.png)
![QQ图片20210922102518.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210922102518-7d91196260e34dba9134ffe77cf85f44.png)
![QQ图片20210922102522.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210922102522-8ec5c2418d2d4f0f96b05beb94939b97.png)
![QQ图片20210922102525.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210922102525-6ed80119cae2490dba879d2d7986fa18.png)

**示例**
> **person的domain对象**
```java
public class Person {
    private int age;
    private String name;
    private BigDecimal money;
    private Dog dog;
    private Set<String> nickNames;
    private Properties friends;

    public Person() {
        System.out.println("Person()");
    }

    public Person(int age) {
        this.age = age;
        System.out.println("Person(" + age + ")");
    }

    @ConstructorProperties({"age", "name"})
    public Person(int age, String name) {
        this.age = age;
        this.name = name;
        System.out.println("Person(" + age + ", " + name + ")");
    }

    public void setFriends(Properties friends) {
        this.friends = friends;
    }

    public Properties getFriends() {
        return friends;
    }

    public void setNickNames(Set<String> nickNames) {
        this.nickNames = nickNames;
    }

    public Set<String> getNickNames() {
        return nickNames;
    }

    public void setDog(Dog dog) {
        this.dog = dog;
    }

    public void setMoney(BigDecimal money) {
        this.money = money;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Person{" +
                "age=" + age +
                ", dog=" + dog +
                ", money=" + money +
                ", name='" + name + '\'' +
                '}';
    }
}

```

> **配置文件applicationContext.xml进行依赖注入**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    
    <bean id="person" class="com.ws.domain.Person"
          p:name="Jack" p:age="18" p:dog-ref="dog"/>

    <bean id="dog" class="com.ws.domain.Dog"/>

<!--    通过setter方法注入-->
<!--    <bean id="person" class="com.ws.domain.Person">-->
<!--        <property name="age" value="18"/>-->
<!--        <property name="money" value="100.5"/>-->
<!--        <property name="name" value="Jack"/>-->
<!--        <property name="dog">-->
<!--            <bean class="com.ws.domain.Dog"/>-->
<!--        </property>-->

<!--        <property name="friends">-->
<!--            <props>-->
<!--                <prop key="Jack">杰克</prop>-->
<!--                <prop key="Rose">螺丝</prop>-->
<!--            </props>-->
<!--        </property>-->
<!--    </bean>-->

<!--    通过构造方法注入-->
<!--    <bean id="person" class="com.ws.domain.Person">-->
<!--        <constructor-arg value="jack" name="name"/>-->
<!--        <constructor-arg value="28" name="age"/>-->
<!--    </bean>-->

</beans>
```

**注意**：
- 基于setter的注入需要先在根标签加一个命名空间属性，其标签大致为 `name`，`value`；
- 基于constructor方法需要先在对应domain(以前的bean)中添加构造方法，其标签大致为`constructor-arg`；

## **Spring 复杂对象**

> 对于复杂对象的创建主要有两种方法：实例工厂和静态工厂，FactoryBean

### **静态工厂步骤：**
①在配置文件中配置调用调用 `ConnectionFactory.getConn()` 方法；
```xml
     静态工厂方法（调用ConnectionFactory.getConn()）
    <bean id="conn" class="com.ws.obj.ConnectionFactory" factory-method="getConn"/>
```

②编写`ConnectionFactory`类和`getConn()`方法，方法中包含具体参数；
```java
public class ConnectionFactory {
    private String driverClass;
    private String url;
    private String username;
    private String password;

    public void setDriverClass(String driverClass) {
        this.driverClass = driverClass;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Connection getConn() throws Exception {
        Class.forName(driverClass);
        return DriverManager.getConnection(url, username, password);
    }
}
```
③在测试类中，先加载配置文件，用该配置文件调用`getBean()`方法，由其中的类名调`getConn()`方法创建对象；
```java
    @Test
    public void test() {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        System.out.println(ctx.getBean("conn"));
    }

```
④静态工厂是用类名去调静态方法；

### **实例工厂**
> **实例工厂将具体参数放在配置文件中，以方便能够动态修改，所以getConn()方法中的参数在xml配置中就已经被动态设置；**

```xml
<!--     实例工厂方法（调用factory.getConn()）-->
    <bean id="factory" class="com.ws.obj.ConnectionFactory">
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/test_mybatis"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
    </bean>
```

### **FactoryBean**
> **第三种方法直接具体实现FactoryBean的getObject方法，其他大致相同；**


```java
public class ConnectionFactoryBean implements FactoryBean<Connection> {
    private String driverClass;
    private String url;
    private String username;
    private String password;

    public void setDriverClass(String driverClass) {
        this.driverClass = driverClass;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public Connection getObject() throws Exception {
        Class.forName(driverClass);
        return DriverManager.getConnection(url, username, password);
    }

    @Override
    public Class<?> getObjectType() {
        return Connection.class;
    }
}
```

### **SpEL表达式**
> **JSP中的EL表达式是用${},而Spring的EL表达式使用#{}**

### **scope**
> **可以通过scope属性控制bean是否单例：其属性singleton单例：通过同一个id值，在同一个IoC容器中获取的永远是同一个实例，所以若创建了其他的IoC容器，通过getBean方法拿到的就不是同一个实例了**

> **另一个属性prototype非单例：每次getBean时创建一次bean**

> **注意：这里的单例不是平时涉及的单例设计模式；同一个容器就是在一个加载配置文件中：new ClassPathXmlApplicationContext("applicationContext.xml")**

**注意：** 为了是配置文件更加简洁，一般需要在db.properties单独配置数据库信息，而数据库的主要属性为：driverClass、username、password、url等，而我们不是直接配置这个属性名，需要在前面加特定字符，以用来区分本机自有属性：jdbc.username而非username；

## **Converter**
1. Spring一般内置了转换器`converter`，如String转int：name="age" value="25", String转Date：name="birthday" value="1997/11/17"
2. 内置转换格式较少，所以需要自定义converter来实现特殊转换格式；
3. 如何自定义？
 ①实现converter接口，在构造format参数后实现convert方法，具体实现转换方法
```java
//日期转换器（String->Date类型）
public class DateConverter implements Converter<String, Date> {
    private List<String> formats;

    public void setFormats(List<String> formats) {
        this.formats = formats;
    }

    @Override
    public Date convert(String s) {
        for (String format : formats) {
            try {
                SimpleDateFormat fmt = new SimpleDateFormat(format);
                return fmt.parse(s);
            } catch (ParseException e) {
//                 e.printStackTrace();
            }
        }
        return null;
    }
}
```
②在applicationContext.xml注册converter，在此表明多种转换格式
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

<!--    这种是使用内置的日期转换器，但是支持格式较少：09/10/2111; 而09_10_2111就不能转换成功-->
<!--    <bean id="person" class="com.ws.domain.Person" p:birthday="2021-06-07"/>-->
    <bean id="person" class="com.ws.domain.Person" p:birthday="06_07_2021"/>

<!--    所以需要自己实现转换-->
    <!-- 配置FactoryBean -->
    <bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <bean class="com.ws.converter.DateConverter">
                    <property name="formats">
                        <list>
                            <value>yyyy-MM-dd</value>
                            <value>MM_dd_yyyy</value>
                        </list>
                    </property>
                </bean>
            </set>
        </property>
    </bean>

</beans>
```


## **Spring bean**

### **什么是 bean？**

简单来说，bean 代指的就是那些被 IoC 容器所管理的对象。

我们需要告诉 IoC 容器帮助我们管理哪些对象，这个是通过配置元数据来定义的。配置元数据可以是 XML 文件、注解或者 Java 配置类。

```xml
<!-- Constructor-arg with 'value' attribute -->
<bean id="..." class="...">
   <constructor-arg value="..."/>
</bean>
```

下图简单地展示了 IoC 容器如何使用配置元数据来管理对象。

![](https://img-blog.csdnimg.cn/062b422bd7ac4d53afd28fb74b2bc94d.png)

`org.springframework.beans`和 `org.springframework.context` 这两个包是 IoC 实现的基础，如果想要研究 IoC 相关的源码的话，可以去看看

### **bean 的作用域有哪些?**

Spring 中 Bean 的作用域通常有下面几种：

- **singleton** : 唯一 bean 实例，Spring 中的 bean 默认都是单例的，对单例设计模式的应用。
- **prototype** : 每次请求都会创建一个新的 bean 实例。
- **request** : 每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP request 内有效。
- **session** : 每一次来自新 session 的 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP session 内有效。
- **global-session** ： 全局 session 作用域，仅仅在基于 portlet 的 web 应用中才有意义，Spring5 已经没有了。Portlet 是能够生成语义代码(例如：HTML)片段的小型 Java Web 插件。它们基于 portlet 容器，可以像 servlet 一样处理 HTTP 请求。但是，与 servlet 不同，每个 portlet 都有不同的会话。

**如何配置 bean 的作用域呢？**

**xml 方式：**

```xml
<bean id="..." class="..." scope="singleton"></bean>
```

**注解方式：**

```java
@Bean
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public Person personPrototype() {
    return new Person();
}
```

### **单例 bean 的线程安全问题了解吗？**

大部分时候我们并没有在项目中使用多线程，所以很少有人会关注这个问题。单例 bean 存在线程问题，主要是因为当多个线程操作同一个对象的时候是存在资源竞争的。

常见的有两种解决办法：

1. 在 bean 中尽量避免定义可变的成员变量。
2. 在类中定义一个 `ThreadLocal` 成员变量，将需要的可变成员变量保存在 `ThreadLocal` 中（推荐的一种方式）。

不过，大部分 bean 实际都是无状态（没有实例变量）的（比如 Dao、Service），这种情况下， bean 是线程安全的。

### **@Component 和 @Bean 的区别是什么？**

1. `@Component` 注解作用于类，而`@Bean`注解作用于方法。
2. `@Component`通常是通过类路径扫描来自动侦测以及自动装配到 Spring 容器中（我们可以使用 `@ComponentScan` 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。`@Bean` 注解通常是我们在标有该注解的方法中定义产生这个 bean,`@Bean`告诉了 Spring 这是某个类的实例，当我需要用它的时候还给我。
3. `@Bean` 注解比 `@Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册 bean。比如当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现。

`@Bean`注解使用示例：

```java
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }

}
```

上面的代码相当于下面的 xml 配置

```xml
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```

下面这个例子是通过 `@Component` 无法实现的。

```java
@Bean
public OneService getService(status) {
    case (status)  {
        when 1:
                return new serviceImpl1();
        when 2:
                return new serviceImpl2();
        when 3:
                return new serviceImpl3();
    }
}
```

### **将一个类声明为 bean 的注解有哪些?**

我们一般使用 `@Autowired` 注解自动装配 bean，要想把类标识成可用于 `@Autowired` 注解自动装配的 bean 的类,采用以下注解可实现：

- `@Component` ：通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注。
- `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。
- `@Controller` : 对应 Spring MVC 控制层，主要用户接受用户请求并调用 Service 层返回数据给前端页面。

### **bean 的生命周期?**

> 下面的内容整理自：<https://yemengying.com/2016/07/14/spring-bean-life-cycle/> ，除了这篇文章，再推荐一篇很不错的文章 ：<https://www.cnblogs.com/zrtqsk/p/3735273.html> 。

- Bean 容器找到配置文件中 `Spring Bean` 的定义。
- Bean 容器利用 `Java Reflection API` 创建一个 `Bean` 的实例。
- 如果涉及到一些属性值 利用 `set()`方法设置一些属性值。
- 如果 Bean 实现了 `BeanNameAware` 接口，调用 `setBeanName()`方法，传入 Bean 的名字。
- 如果 Bean 实现了 `BeanClassLoaderAware` 接口，调用 `setBeanClassLoader()`方法，传入 `ClassLoader`对象的实例。
- 如果 Bean 实现了 `BeanFactoryAware` 接口，调用 `setBeanFactory()`方法，传入 `BeanFactory`对象的实例。
- 与上面的类似，如果实现了其他 `*.Aware`接口，就调用相应的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessBeforeInitialization()` 方法
- 如果 Bean 实现了`InitializingBean`接口，执行`afterPropertiesSet()`方法。
- 如果 Bean 在配置文件中的定义包含 init-method 属性，执行指定的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessAfterInitialization()` 方法
- 当要销毁 Bean 的时候，如果 Bean 实现了 `DisposableBean` 接口，执行 `destroy()` 方法。
- 当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含 `destroy-method` 属性，执行指定的方法。

图示：

![Spring Bean 生命周期](https://images.xiaozhuanlan.com/photo/2019/24bc2bad3ce28144d60d9e0a2edf6c7f.jpg)

与之比较类似的中文版本:

![Spring Bean 生命周期](https://images.xiaozhuanlan.com/photo/2019/b5d264565657a5395c2781081a7483e1.jpg)

**总结：**
> **生命周期就是bean从出生到死亡经历的周期方法：**

```java
      # 构造方法
      01 - UserServiceImpl

      # 属性输入:setter
      02 - setAge - 20

      # 让你知道一下bean的名字（id、name）:BeanNameAware的setBeanName
      03 - BeanNameAware - setBeanName - userService

      # 让你知道一下你在哪个容器里面:ApplicationContextAware的setApplicationContext
      04 - ApplicationContextAware - setApplicationContext - org.springframework.context.support.ClassPathXmlApplicationContext@14acaea5, started on Fri Aug 21 21:14:50 CST 2020

      # 初始化方法调用之前调用
      05 - BeanPostProcessor - postProcessBeforeInitialization - userService

      # 构造、注入完毕之后调用①（初始化，加载资源）
      06 - InitializingBean - afterPropertiesSet

      # 构造、注入完毕之后调用②（初始化，加载资源）
      07 - init-method

      # 初始化方法调用之后调用
      08 - BeanPostProcessor - postProcessAfterInitialization - userService

      # 业务方法
      09 - UserServiceImpl - login - 123_456

      # 销毁之前调用①（释放资源）
      10 - DisposableBean - destroy

      # 销毁之前调用②（释放资源）
      11 - destroy-method
```

> **注意：第5、8两步是在初始化资源加载前后执行，所以可以在这两步对其进行拦截做一些判断和处理操作，只有当bean的scope为singleton时，IoC容器关闭后才会调用第10、11个方法。测试步骤如下：**

**1. 在UsreService创建业务方法login()**
```java
public interface UserService {
    boolean login(String username, String password);
}
```
**2. 在UserServiceImpl中具体实现业务方法，同时测试bean的生命周期，注意实现的哪些类和需要注入的属性age**
```java
public class UserServiceImpl implements
        UserService,
        BeanNameAware, ApplicationContextAware,
        InitializingBean, DisposableBean {

    private Integer age;

    // 1、构造方法为第一个调用的
    public UserServiceImpl() {
        System.out.println("01 - UserServiceImpl");
    }

    // 2、setter为第二个调用的生命周期
    public void setAge(Integer age) {
        this.age = age;
        System.out.println("02 - setAge - " + age);
    }

    // 3、实现BeanNameAware接口：让你知道一下bean的名字（id/name：userService/setBeanName）
    @Override
    public void setBeanName(String name) {
        System.out.println("03 - BeanNameAware - setBeanName - " + name);
    }

    // 4、ApplicationContextAware接口：让你知道一下你在哪个容器里面(setApplicationContext)
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("04 - ApplicationContextAware - setApplicationContext - " + applicationContext);
    }

    // 6、InitializingBean：构造、注入完毕之后调用①（初始化，加载资源：afterPropertiesSet）
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("06 - InitializingBean - afterPropertiesSet");
    }

    // 7、构造、注入完毕之后调用②（初始化，加载资源）
    public void init() {
        System.out.println("07 - init-method");
    }

    // 9、UserServiceImpl：login这个业务方法为第九个被调用的
    @Override
    public boolean login(String username, String password) {
        System.out.println("09 - UserServiceImpl - login - " + username + "_" + password);
        return false;
    }

    // 10、DisposableBean：销毁之前调用①（释放资源）
    @Override
    public void destroy() throws Exception {
        System.out.println("10 - DisposableBean - destroy");
    }

    // 11、销毁之前调用②（释放资源）
    public void dealloc() {
        System.out.println("11 - destroy-method");
    }
}
```

**3. 在xml中注入bean对象，这里只有一个age属性**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userService"
          class="com.ws.service.impl.UserServiceImpl"
          p:age="20"
          init-method="init"
          destroy-method="dealloc"/>

    <bean class="com.ws.processor.MyProcessor" scope="singleton"/>
</beans>
```

**4. 上面可知，11个bean生命周期并未测试完全，所以5,8需要单独测试且可以统一处理所有的Bean**
```java
/**
 * 可以统一处理所有的Bean
 */
public class MyProcessor implements BeanPostProcessor {
    // 5、初始化方法调用之前调用BeanPostProcessor - postProcessBeforeInitialization
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("05 - BeanPostProcessor - postProcessBeforeInitialization - " + beanName);
        return bean;
    }

    // 8、初始化方法调用之后调用:BeanPostProcessor - postProcessAfterInitialization
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("08 - BeanPostProcessor - postProcessAfterInitialization - " + beanName);
        return bean;
    }
}
```

**5. 测试类**
```java
public class UserTest {
    @Test
    public void test() {
        // 创建容器
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");

        UserService service = ctx.getBean("userService", UserService.class);
        service.login("123", "456");

        // 关闭容器
        ctx.close();
    }
```

**6. 测试结果：确实是顺序执行的**

![QQ图片20210922170748.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210922170748-6d5f906c0fd44500aee2c600508ed7a5.png)

## **Spring 代理**
> **业务层service中除了业务代码，如业务运算和dao操作等必要的操作，还有附加的代码，如事务、日志、性能监控、异常处理等可选的操作；**

> **所以附加代码的加入就导致了业务层代码的累赘，这里就提出了代理：就是在网络层调用代理类。代理类包含业务代码和附加代码，通过业务代码调用业务层的业务代码；**

> **代理就是在不修改目标类的目标方法代码的前提下，为目标方法增加额外功能；代理类中必须也有同样的目标方法，即 代理类实现跟目标类同样的接口，若目标类没有实现接口，代理类继承目标类；**

> **代理分为静态和动态，其中动态代理可以是JDK自带的（目标类必须有具体实现）和 CGLib（可以没有实现类），具体见Java重要知识点2！**

## **Spring AOP**

### **谈谈自己对于 AOP 的了解**

AOP (Aspect-Oriented Programming:面向切面编程) 能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。

Spring AOP 就是基于动态代理的，如果要代理的对象，实现了某个接口，那么 Spring AOP 会使用 **JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候 Spring AOP 会使用 **Cglib** 生成一个被代理对象的子类来作为代理，如下图所示：

![SpringAOPProcess](https://images.xiaozhuanlan.com/photo/2019/926dfc549b06d280a37397f9fd49bf9d.jpg)

当然你也可以使用 **AspectJ** ！Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。

### **Spring AOP 和 AspectJ AOP 有什么区别？**

**Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。** Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。

Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，

如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比 Spring AOP 快很多

### **具体实现**
> **通过AOP完成附加功能主要有两个方法：实现 `MethodBeforeAdvice` 接口（只会在目标方法执行前执行附加代码）以及实现 `MethodInterceptor` 接口（前后均可）；**

**实现 `MethodBeforeAdvice` 接口**

1. 配置applicationContext.xml文件（配置附加代码执行位置以及切面）
```xml
    <!-- 附加代码执行处 -->
    <bean id="logAdvice" class="com.ws.aop.LogAdvice"/>

    <aop:config>
        <!-- 切入点：给哪些类的哪些方法增加附加代码？ -->
        <!-- execution(* *(..))代表所有类的所有方法都会被切入 -->
        <aop:pointcut id="pc" expression="execution(* *(..))"/>

        <!-- 附加代码位置 -->
        <aop:advisor advice-ref="logAdvice" pointcut-ref="pc"/>
    </aop:config>
```
2. 附加代码类
```java
public class LogAdvice implements MethodBeforeAdvice {
    @Override
    public void before(Method method, Object[] args, Object target) throws Throwable {
        System.out.println("LogAdvice - before ------------------");
    }
}
```
3. 在各自的接口或者实现了接口的具体实现类中，编写方法（注解`@Log`是自定义注解的方法`@annotation`切入），省略`userService/Impl`和`personService/Impl`。
```java
// 只有接口，没有实现类（使用CGLib代理），而如果由实现类，使用JDK自带的代理
public class SkillService {
    private int age;
    @Log
    public boolean save(Object skill) {
        System.out.println("SkillService - save");
        return false;
    }
}
```

4. 测试代码
```java
// 某些类的某些方法才需要代理去增加附加代码
public class UserTest {

    @Test
    public void test() {
        // 创建容器
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");

        SkillService skillService = ctx.getBean("skillService", SkillService.class);
        skillService.save(null);

        UserService userService = ctx.getBean("userService", UserService.class);
        userService.login(null, null);
        userService.register(null);

        PersonService personService = ctx.getBean("personService", PersonService.class);
        personService.run();
    }
}
```
5. 测试结果
![QQ图片20210922224954.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210922224954-48ac3d75ca3341e385ecfdfec35e457e.png)

> **注意:** 所有接口的所有方法都像在配置文件中设置的那样，全部接口的全部方法都会被AOP附加代码。
> **但是存在一个缺点就是附加代码只能在目标方法的前面！**


**实现 `MethodInterceptor` 接口**

1. 配置applicationContext.xml文件（配置附加代码执行位置以及切面）
```xml
    <!-- 附加代码执行处 -->
    <bean id="logInterceptor" class="com.ws.aop.LogInterceptor"/>


    <aop:config proxy-target-class="false">
        <!-- 切入点：给哪些类的哪些方法增加附加代码？ -->
        <!-- execution(* *(..))代表所有类的所有方法都会被切入 -->
        <aop:pointcut id="pc" expression="execution(* *(..))"/>

        <!-- 附加代码位置 -->
        <aop:advisor advice-ref="logInterceptor" pointcut-ref="pc"/>
    </aop:config>
```

2. 附加代码类
```java
public class LogInterceptor implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {

        System.out.println("1 ----------------");

        // 调用目标对象的目标方法
        Object result = invocation.proceed();

        System.out.println("2 ----------------");

        return result;
    }
}
```
3. 在各自的接口或者实现了接口的具体实现类中，编写方法，省略`skillService`和`personService/Impl`。
```java
// 定义接口
public interface UserService {
    boolean login(String username, String password);
    boolean register(String username);
}
```
```java
// 具体实现类
public class UserServiceImpl implements UserService, BeanNameAware, ApplicationContextAware {
    private ApplicationContext ctx;
    private String beanName;

    @Override
    public boolean login(String username, String password) {
        System.out.println("UserServiceImpl - login");
        return false;
    }

    @Override
    @Log
    public boolean register(String username) {
        System.out.println("UserServiceImpl - register");
        return false;
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        ctx = applicationContext;
    }

    @Override
    public void setBeanName(String name) {
        beanName = name;
    }
}
```
4. 测试代码相同，测试结果如下：

![QQ图片20210923093953.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210923093953-63045448c02241b4934de0541a812686.png)

> **由结果可知：实现了AOP，而且在目标方法前后都可以附加代码**

**AOP动态代理的底层实现**
- Spring动态代理的底层实现：如果果目标类有实现接口，使用 `JDK` 实现；如果目标类没有实现接口，使用 `CGLib` 实现。
- 在 `BeanPostProcessor` 的 `postProcessAfterInitialization` 方法中创建代理对象。
- 也可以通过 `proxy-target-class` 属性修改底层实现方案：true（强制使用CGLib），false（按照默认方法）。
```xml
	<aop:config proxy-target-class="false">
```

**AOP切入点表达式**
> **常见的切入点指示符有：`execution`、`args`、`within`、`@annotation` 等，当然了还可以使用 `&&(and)`、`||(or)`、`！(非)` 组合切入点表达式！**

![QQ图片20210923100532.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210923100532-7a9767925a3647899bfdf0d0372e5c20.png)

**注意：** 默认情况下，目标方法相互调用时，总共只会被切入1次。解决方法：拿到代理对象去调用目标方法，见UserServiceImpl

> **当然了，AOP可以同时配置多个`pointcut`、`advisor`！**
```xml
    <!-- 附加代码执行处 -->
    <bean id="logAdvice" class="com.ws.aop.LogAdvice"/>
    <bean id="logInterceptor" class="com.ws.aop.LogInterceptor"/>

    <!--配置多个切面的第一种方式-->
    <aop:config>
        <aop:pointcut id="pc1" expression="within(com.ws.service.impl.UserServiceImpl)"/>
        <aop:advisor advice-ref="logInterceptor" pointcut-ref="pc1"/>
        <aop:advisor advice-ref="logAdvice" pointcut-ref="pc1"/>
    </aop:config>

    <aop:config>
        <aop:pointcut id="pc2" expression="within(com.ws.service.SkillService)"/>
        <aop:advisor advice-ref="logInterceptor" pointcut-ref="pc2"/>
    </aop:config>

    <!--配置多个切面的第一种方式-->
    <aop:config>
        <aop:pointcut id="pc1" expression="within(com.ws.service.impl.UserServiceImpl)"/>
        <aop:pointcut id="pc2" expression="within(com.ws.service.SkillService)"/>

        <aop:advisor advice-ref="logInterceptor" pointcut-ref="pc1"/>
        <aop:advisor advice-ref="logInterceptor" pointcut-ref="pc2"/>
        <aop:advisor advice-ref="logAdvice" pointcut-ref="pc1"/>
    </aop:config>
```

**总结：** 
- AOP面向切面编程: Spring使用AOP技术封装了动态代理的功能，就不用像上节那样创建代理；
- 切面：包含切入点 `aop:pointcut`，即给哪些类的哪些方法增加附加代码; 也包含附加代码的位置`aop:advisor`， 切到切入点对应方法后，在AOP技术下，并没有了创建代理对象和invoke()回调；
- `LogAdvice`中实现了`MethodBeforeAdvice`，只有一个`before`方法，即只能在切入点的前面插入代码；
- 而`LogInterceptor`实现了`MethodInterceptor`，只需实现它的`invoke方法`就能在前后插入代码了；
- 由本节可知，创建动态代理的方法有两种，一是JDK自带的，二是Spring集成的CGLib，其创建代理同样也是在 `Bean` 生命周期的第八步 `BeanPostProcessor` 的 `postProcessAfterInitialization` 方法中创建；
- 切面配置里面，附加代码映射位置相对简单，而具体切入点使用到了AOP切入点表达式! 常见的切入点指示符: `execution/args/within/@annotation`，具体看参考资料；
 ①`@annotation`：`@annotation(com.ws.annotation.Log)`: 表示注解中有Log注解的方法进行切入。
 ②`AOP`细节：默认情况下，目标方法相互调用时，总共只会被切入1次，解决方法是：拿到代理对象去调用目标方法。
 ③`within`：`within(com.ws.service.impl.UserServiceImpl)`: 表示切入了UserServiceImpl里的全部方法。

## **Spring 事务**

Spring/SpringBoot 模块下专门有一篇是讲 Spring 事务的，总结的非常详细，通俗易懂。

### **Spring 管理事务的方式有几种？**

- **编程式事务**： 在代码中硬编码(不推荐使用) : 通过 `TransactionTemplate`或者 `TransactionManager` 手动管理事务，实际应用中很少使用，但是对于你理解 Spring 事务管理原理有帮助。
- **声明式事务**： 在 XML 配置文件中配置或者直接基于注解（推荐使用），实际是通过 AOP 实现（基于`@Transactional` 的全注解方式使用最多），使用Spring的声明式事务，可以极其容易地进行事务管理（只需要在 `XML` 中进行声明配置，一般是在 `Service` 层进行事务管理）。

### **Spring 事务中哪几种事务传播行为?**

**事务传播行为是为了解决业务层方法之间互相调用的事务问题**。

当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。

正确的事务传播行为可能的值如下:

**1.`TransactionDefinition.PROPAGATION_REQUIRED`**

使用的最多的一个事务传播行为，我们平时经常使用的`@Transactional`注解默认使用就是这个事务传播行为。如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。

**`2.TransactionDefinition.PROPAGATION_REQUIRES_NEW`**

创建一个新的事务，如果当前存在事务，则把当前事务挂起。也就是说不管外部方法是否开启事务，`Propagation.REQUIRES_NEW`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。

**3.`TransactionDefinition.PROPAGATION_NESTED`**

如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于`TransactionDefinition.PROPAGATION_REQUIRED`。

**4.`TransactionDefinition.PROPAGATION_MANDATORY`**

如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）

这个使用的很少。

若是错误的配置以下 3 种事务传播行为，事务将不会发生回滚：

- **`TransactionDefinition.PROPAGATION_SUPPORTS`**: 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- **`TransactionDefinition.PROPAGATION_NOT_SUPPORTED`**: 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- **`TransactionDefinition.PROPAGATION_NEVER`**: 以非事务方式运行，如果当前存在事务，则抛出异常。

### **Spring 事务中的隔离级别有哪几种?**

和事务传播行为这块一样，为了方便使用，Spring 也相应地定义了一个枚举类：`Isolation`

```java
public enum Isolation {

    DEFAULT(TransactionDefinition.ISOLATION_DEFAULT),

    READ_UNCOMMITTED(TransactionDefinition.ISOLATION_READ_UNCOMMITTED),

    READ_COMMITTED(TransactionDefinition.ISOLATION_READ_COMMITTED),

    REPEATABLE_READ(TransactionDefinition.ISOLATION_REPEATABLE_READ),

    SERIALIZABLE(TransactionDefinition.ISOLATION_SERIALIZABLE);

    private final int value;

    Isolation(int value) {
        this.value = value;
    }

    public int value() {
        return this.value;
    }

}
```

下面我依次对每一种事务隔离级别进行介绍：
> **`TransactionDefinition.ISOLATION_DEFAULT`** :使用后端数据库默认的隔离级别，MySQL 默认采用的 `REPEATABLE_READ` 隔离级别 Oracle 默认采用的 `READ_COMMITTED` 隔离级别，而Isolution用于设置事务的隔离级别。
- **`TransactionDefinition.ISOLATION_READ_UNCOMMITTED`** :最低的隔离级别，使用这个隔离级别很少，因为它允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**
- **`TransactionDefinition.ISOLATION_READ_COMMITTED`** : 允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**
- **`TransactionDefinition.ISOLATION_REPEATABLE_READ`** : 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。**
- **`TransactionDefinition.ISOLATION_SERIALIZABLE`** : 最高的隔离级别，完全服从 ACID 的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

### **@Transactional(rollbackFor = Exception.class)注解了解吗？**

`Exception` 分为运行时异常 `RuntimeException` 和非运行时异常。事务管理对于企业应用来说是至关重要的，即使出现异常情况，它也可以保证数据的一致性。

当 `@Transactional` 注解作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。如果类或者方法加了这个注解，那么这个类里面的方法抛出异常，就会回滚，数据库里面的数据也会回滚。

在 `@Transactional` 注解中如果不配置`rollback-for`属性,那么事务只会在遇到`RuntimeException`的时候才会回滚，加上 `rollback-for=Exception.class`,可以让事务在遇到非运行时异常时也回滚。

### 具体如下：
```xml
    <!--引入数据库配置文件db.properties-->
    <context:property-placeholder location="db.properties"/>

    <!-- 将数据库配置文件属性设置到数据源（Druid） -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driverClass}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <!-- 创建SqlSessionFactory -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 数据源 -->
        <property name="dataSource" ref="dataSource"/>
        <!-- 这个包底下的类会自动设置别名（一般是领域模型） -->
        <property name="typeAliasesPackage" value="com.ws.domain"/>
        <!-- 映射文件的位置 -->
        <property name="mapperLocations">
            <array>
                <value>mappers/*.xml</value>
            </array>
        </property>
    </bean>

    <!-- 扫描dao -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!-- 设置SqlSessionFactoryBean的id -->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <!-- 设置dao的包 -->
        <property name="basePackage" value="com.ws.dao"/>
    </bean>

    <bean id="skillService" class="com.ws.service.impl.SkillServiceImpl">
        <property name="dao" ref="skillDao"/>
    </bean>

    <!-- 事务管理器：通过AOP实现 -->
    <bean id="txMgr"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 附加代码：事务管理代码 -->
    <tx:advice id="txAdvice" transaction-manager="txMgr">
        <tx:attributes>
<!--        no-rollback-for这里表示哪些异常不会导致事务回滚；rollback-for哪些异常会导致事务回滚-->
            <tx:method name="*" no-rollback-for="RuntimeException"/>
            <tx:method name="list*" propagation="SUPPORTS"/>
            <tx:method name="update*"/>
        </tx:attributes>
    </tx:advice>

    <!-- 切面 -->
    <aop:config>
        <aop:pointcut id="pc" expression="within(com.ws.service..*)"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pc"/>
    </aop:config>
```

**由上可知：**
> **propagation**

可以通过 `propagation` 属性设置事务的传播行为。用于指定当事务嵌套时，如何管理事务，当Service调用Service时，会出现事务嵌套，注意各传播行为的使用场景。

![QQ图片20210923104836.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210923104836-61d6d8b21a1f411da92a5587a8f41bac.png)

```xml
	<!--使用xml设置-->
	<tx:method name="list*" propagation="SUPPORTS"/>
```

> **read-only**

如果一个事务只执行读操作，那么数据库可能会采取某些优化措施：`read-only设置为true`，告诉数据库这是个只读事务（只适用于 `REQUIRED` 或者 `REQUIRED_NEW`）
```XML
	<tx:method name="list*" propagation="REQUIRED" read-only="true"/>
```

> **timeout**

单位是秒，默认是-1（按照数据库默认情况处理），超时就会抛出异常
```XML
	<tx:method name="list*" timeout="4"/>
```

> **rollback-for、no-rollback-for**

默认情况下，`RuntimeException`、`Error`会导致事务回滚，而 `Exception` 不会；
`rollback-for:` 设置哪些异常会导致事务回滚，即在上述RuntimeException、Error两个异常基础上增加异常；`no-rollback-for:` 设置哪些异常不会导致事务回滚，即在Exception基础上增加一些异常；
```xml
	<!--本来RuntimeException回导致事务回滚，下面设置之后便不会回滚-->
	<tx:method name="*" no-rollback-for="RuntimeException"/>
```

> **isolation**
 isolation用于设置事务的隔离级别，照数据库的默认设置级别为 `REPEATABLE_READ`，略！详见Spring事务总结
```sql
	# 查询隔离级别的SQL语句
	SELECT @@TX_ISOLATION

	# 设置隔离级别
	SET GLOBAL TRANSACTION ISOLATION LEVEL 隔离级别
```

## **Spring 注解**

> **提示：如果Spring的配置文件内容过多，可以采取多个配置文件的形式，可以在加载配置文件时依次加载或通配符加载，或者在其中一个配置文件中import导入其他几个。**
```java
public class MyTest {
    @Test
    public void test() {
        // 引入配置文件：分看单独写
        ApplicationContext ctx = new ClassPathXmlApplicationContext(
                "applicationContext.xml",
                "applicationContext-mybatis.xml",
                "applicationContext-tx.xml");
        // 用通配符
//        ApplicationContext ctx = new ClassPathXmlApplicationContext(
//                "classpath*:applicationContext*.xml");
        System.out.println(ctx.getBean("person"));
        System.out.println(ctx.getBean("dog"));
        System.out.println(ctx.getBean("user"));
    }
}
```

### **1.注解实现bean标签**

- `@Component`：相当于`<bean>`；通过value属性设置bean的id，默认会将类名的首字母小写形式作为bean的id；`@Controller`用于标识控制器层，`@Service`用于标识业务层，`@Repository`用于标识Dao层（前面三者都是用于配置文件的扫描，只是名称不一样，只是区分哪层，如果不知道哪层就使用`@Component`）。
- `@Scope`：设置`singleton（单列）`、`prototype`。
- `@Lazy`：scope为singleton时延迟加载。
- 可以通过`<context:component-scan>`设置需要扫描的包，可以扫描到所有的`@Component`，其中`@ComponentScan`相当于`<context:component-scan>`
```xml
	<context:component-scan base-package="com.ws"/>
```

### **2.注解实现注入**

- `@Autowired`：默认按照类型注入bean实例；可以写在成员变量（不会调用seeter）、setter 、构造方法上；可以配合使用`@Qualifier`、`@Named`：设置需要注入的bean的id；required设置为false时表示找不到对应bean抛出异常。
- `@value`: 用于注入String、基本类型、BigDecimal等类型，可以配合配置文件，使用${}读取配置文件中的内容。
- `@PropertySource`、`@PropertySources`相当于 
```<context:property-placeholder location="user.properties"/>```

bean注入对象：
```java
@Component
@PropertySource("user.properties")
public class User {
    @Value("${name}")
    private String name;
    private Dog dog;

    public void setName(String name) {
        this.name = name;
    }

    @Autowired(required = false)
    @Qualifier("dog")
    public void setDog(Dog dog) {
        this.dog = dog;
    }

    @Override
    public String toString() {
        return "User{" +
                "name=" + name +
                ",dog=" + dog +
                '}';
    }
}
```
配置文件user.properties：
```xml
	name=Rose
```

### **3.注解实现注入构造方法**

可以通过`@Autowired`指定创建bean时调用的构造方法
![QQ图片20210923161612.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210923161612-26d177efe9cb4b6e80582fb97756b4f8.png)

### **4.注解实现AOP**

- `@Aspect`: 用来标识一个切面类，其对象需要放到IoC容器中
- `@Around`: 用来设置切入的附加代码
- 还需要加上 `<aop:aspectj-autoproxy>` 标签，替代之前的 `<aop:config>`：同样有 `proxy-target-class` 属性设置AOP的底层实现方案
- `@Pointcut`复用切入点，即用EL表达式表示哪个类中的哪个方法切入

```java
// 切面类
@Aspect
@Component
public class DefaultAspect {
    // 切入的目标
    @Pointcut("within(com.ws.service.impl.UserServiceImpl)")
    public void pc() {}

    // 切入点的附加代码
    @Around("pc()")
    public Object log(ProceedingJoinPoint point) throws Throwable {
        System.out.println("log-----------------------1");

        // 调用目标方法
        Object ret = point.proceed();

        System.out.println("log-----------------------2");
        return ret;
    }

    @Around("pc()")
    public Object watch(ProceedingJoinPoint point) throws Throwable {
        System.out.println("watch-----------------------1");

        // 调用目标方法
        Object ret = point.proceed();

        System.out.println("watch-----------------------2");
        return ret;
    }
}
```
xml
```xml
    <context:component-scan base-package="com.ws"/>

<!--    这里是用来替代之前的<aop:config>-->
    <aop:aspectj-autoproxy/>
```

### **5.注解实现事务**

- `@Transactional`: 设置在需要进行事务管理的类（一般是service）上使用这个注解,表示所有方法都会自动管理事务。
- 可以在需要进行事务管理的方法上使用这个注解（可以覆盖类中`@Transactional`的配置）。
- 可以设置`isolation`、`propagation` 、`timeout` 、`rollbackFor` 、`noRollbackFor` 、`readOnly` 等前面介绍的属性。
- 使用`<tx:annotation-driven>`取代之前的 `<tx:advice>`、`<aop:config>`，同样有`proxy-target-class`属性设置AOP的底层实现方案。
```java
@Service("skillService")
@Transactional  // 作用到类，表示全部方法都事务管理
public class SkillServiceImpl implements SkillService {
    private SkillDao dao;

    @Autowired
    public void setDao(SkillDao dao) {
        this.dao = dao;
    }

    @Override
    @Transactional(propagation = Propagation.SUPPORTS) // 作用到具体方法，可以覆盖类的@Transactional设置
    public List<Skill> list() {
        return dao.list();
    }

    @Override
    public boolean save(Skill skill) {
        return dao.save(skill);
    }

    @Override
    public boolean update(Skill skill) {
        return dao.update(skill);
    }

    public void test(Skill skill) throws Exception {

        dao.save(skill);

        throw new RuntimeException();
    }
}
```

### **6.@Configuration、@Bean**

- `@Configuration`注解可以取代`applicationContext.xml`,它也是一个`@Component`。
- 可以在`@Configuration`注解的类中，使用`@Bean`修饰方法，进行bean对象的创建，默认情况下方法名就是`bean`的`id`，也可以通过`name`，`value`属性设置`bean`的`id`。
- 如果`bean`属性本身有`@Autowired`，那么直接new原来对象即可，Spring会利用`@Autowired`技术，自动注入`bean`给`@Bean`方法的参数。
- `@Bean`方法的注入`FacoryBean`中，表面上`@Bean`方法返回的是`DogFacoryBean`，但由于`@Configuration`底层使用了`CGLib`，对`@Bean`进行了增强处理，所以会调用`DogFactoryBean`的`getObject`方法。
```java
@Component
public class DogFactoryBean implements FactoryBean<Dog> {
    @Override
    public Dog getObject() throws Exception {
        return new Dog();
    }

    @Override
    public Class<?> getObjectType() {
        return Dog.class;
    }
}
```
- @Bean方法注入其他类型
```java
@Configuration
@PropertySource("skill.properties")
public class BeanConfig {
    @Value("${name}")
    private String name;
    @Value("${level}")
    private int level;

    @Bean
    public Skill skill() {
        Skill skill = new Skill();
        skill.setName(name);
        skill.setLevel(level);
        return skill;
    }
```
配置文件skill.properties：
```xml
name=Rose
level=20
```

### **7.创建工厂的入口**

- 如果创建工厂的入口是XML，一般就用`ClassPathXmlApplicationContext`来加载xml文件；xml文件中可以通过`<context:component-scan>`来扫描注解信息，通过<import>来导入其他xml配置文件；
- 如果创建工厂的入口是注解，一般就用`AnnotationConfigApplicationContext`来加载，可以通过注解`@Import`、`@ComponentScan`扫描其他注解信息，通过`@ImportSource`导入其他注解信息；
```java
	ctx = new AnnotationConfigApplicationContext("com.ws");
```
- 虽然有各种各样的关联扫描方法，但我们可以将各种配置文件写在一个集中的类中：`BeanConfig`
- 在Spring中，bean有3种常见的创建方式：①`@Component`: 常用于源码可修改，创建过程比较简单，直接调用构造方法即可的类型；②`@Bean`: 常用于源码不可修改的类型(如第三方库)或者创建过程比较复杂的类型；③`<Bean>`: xml文件适用于所有类型。
- 使用优先级:`③>②>①`

### **8.纯注解开发AOP**

纯注解开发AOP，利用`@EnableAspectJAutoProxy`代替`<aop:aspectj-autoproxy/>`
```java
@Aspect
@Component
public class DefaultAspect {
    @Pointcut("within(com.ws.service.impl.UserServiceImpl)")
    public void pc() {}

    @Around("pc()")
    public Object log(ProceedingJoinPoint point) throws Throwable {
        System.out.println("log-----------------------1");

        // 调用目标方法
        Object ret = point.proceed();

        System.out.println("log-----------------------2");
        return ret;
    }

    @Around("pc()")
    public Object watch(ProceedingJoinPoint point) throws Throwable {
        System.out.println("watch-----------------------1");

        // 调用目标方法
        Object ret = point.proceed();

        System.out.println("watch-----------------------2");
        return ret;
    }
}
```
单独一个类
```java
@Configuration
@EnableAspectJAutoProxy
@ComponentScan("com.ws")
public class AppConfig {
}

```
这样就可以实现AOP

### **9.纯注解开发整合MyBatis**

```java
// 利用spring的纯注解整合mybatis
@Configuration
@PropertySource("db.properties")
@MapperScan("${mybatis.mapperScan}")
public class MyBatisConfig {
    @Value("${jdbc.driverClassName}")
    private String driverClassName;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String username;
    @Value("${jdbc.password}")
    private String password;
    @Value("${mybatis.typeAliasesPackage}")
    private String typeAliasesPackage;
    @Value("${mybatis.mapperLocations}")
    private String mapperLocations;

    // 数据源
    @Bean
    public DataSource dataSource() {
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName(driverClassName);
        ds.setUrl(url);
        ds.setUsername(username);
        ds.setPassword(password);
        return ds;
    }

    // 创建sqlSessionFactory
    @Bean
    public SqlSessionFactoryBean sqlSessionFactory() throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource());
        bean.setTypeAliasesPackage(typeAliasesPackage); // 这个包下的类自动设置别名
        PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        bean.setMapperLocations(resolver.getResources(mapperLocations)); // 映射文件的位置(db.properties中已配置)
        return bean;
    }

    // 扫描Dao可以省略;直接由@MapperScan("${mybatis.mapperScan}")注解代替
//    @Bean
//    public MapperScannerConfigurer configurer() {
//        MapperScannerConfigurer configurer = new MapperScannerConfigurer();
//        configurer.setSqlSessionFactoryBeanName("sqlSessionFactory");
//        configurer.setBasePackage("com.ws.dao");
//        return configurer;
//    }
```
对比：

![QQ图片20210923171407.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210923171407-143b2988b536450891e9eb9fcff445cd.png)

### **10.纯注解开发事务管理**

纯注解的事务管理，用注解`@EnableTransactionManagement`替代`<tx:annotation-driven>`
```java
@Configuration
@EnableTransactionManagement
public class TxConfig {
    @Bean
    public DataSourceTransactionManager mgr(DataSource dataSource) {
        DataSourceTransactionManager mgr = new DataSourceTransactionManager();
        mgr.setDataSource(dataSource);
        return mgr;
    }
}
```

### **11.JSR注解**
略

### **12.component-scan详解**

![QQ图片20210923171939.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210923171939-31a3550dab0d423c92c39d82115a0967.png)

![QQ图片20210923171943.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210923171943-2f688ce56bb34e8abaca94954676fc90.png)

![QQ图片20210923171946.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210923171946-36040c3034a54305bb67a7fde205cc15.png)

## **JPA**

### **如何使用 JPA 在数据库中非持久化一个字段？**

假如我们有下面一个类：

```java
@Entity(name="USER")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "ID")
    private Long id;

    @Column(name="USER_NAME")
    private String userName;

    @Column(name="PASSWORD")
    private String password;

    private String secrect;

}
```

如果我们想让`secrect` 这个字段不被持久化，也就是不被数据库存储怎么办？我们可以采用下面几种方法：

```java
static String transient1; // not persistent because of static
final String transient2 = "Satish"; // not persistent because of final
transient String transient3; // not persistent because of transient
@Transient
String transient4; // not persistent because of @Transient
```

一般使用后面两种方式比较多，我个人使用注解的方式比较多。

## **参考**

- 《Spring 技术内幕》
- <http://www.cnblogs.com/wmyskxz/p/8820371.html>
- <https://www.journaldev.com/2696/spring-interview-questions-and-answers>
- <https://www.edureka.co/blog/interview-questions/spring-interview-questions/>
- https://www.cnblogs.com/clwydjgs/p/9317849.html
- <https://howtodoinjava.com/interview-questions/top-spring-interview-questions-with-answers/>
- <http://www.tomaszezula.com/2014/02/09/spring-series-part-5-component-vs-bean/>
- <https://stackoverflow.com/questions/34172888/difference-between-bean-and-autowired>