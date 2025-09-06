## **SpingBoot - Maven补充**

### **一、依赖冲突**

![QQ图片20210925161432.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210925161432-2bd4801c2b3f4e1992be661f96a0027c.png)

**如何解决依赖冲突？**

> 方法一：默认情况下：优先保留前面先声明的版本，所以就算出现的依赖冲突但整个项目运行中是不会报错的，Maven已经自动处理好了！

> 方法二：单独为依赖库增加`dependency`指定版本号，就如依赖冲突例子可得，可以直接在dependencies中添加spring-beans的版本！

> 方法三：使用`exclusion`排除某个依赖，比如在添加spring-webmvc依赖时，使用属性标签`exclusion`排除`spring-beans`，这样`spring-beans`版本只能选择`spring-jdbc`的版本了！

> 方法四(最常用的解决方案)：使用`dependencyManagement`锁定依赖库的版本号，注意该方法只能声明版本号，不能真正下载导入依赖库！


### **二、分模块构建项目**
项目规模比较庞大时，可以考虑对项目进行拆分，分模块进行构建项目，有2种常见的拆分思路：按业务模块：员工模块、部门模块、工资模块等；按层：dao层、service层、web层等。

![QQ图片20210925162519.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210925162519-787f1573a7c9403782657c3045ba87fc.png)

**继承**
子项目继承父项目后，可以继承父项目中的`pom.xml`文件中的一些内容：比如`dependencies`、`dependencyManagement`、`properties`等等

> **注意：父项目的`packaging`必须是`pom`，子项目通过`parent标签`继承父项目**

**聚合**
可以通过modules标签聚合多个项目，可以对多个项目进行统一构建管理，通常会在父项目中聚合它的子项目。

![QQ图片20210925162925.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210925162925-40e7cb8d82c249c09fcce77a153ad93d.png)

**依赖**
一个项目A可以使用dependency依赖另一个项目B，项目A中可以直接拥有项目B的classpath中的内容（比如类、配置文件、依赖库等），下面的代码是web项目依赖service项目：

![QQ图片20210925163121.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210925163121-0844d005f5f846f2b076de9008da4259.png)

> **注意：** 区别模块的继承和依赖的区别：继承是用pom.xml的配置，而依赖是用classpath中的内容。

**各模块的分工：**
- 父模块是进行全局定义；
- dao模块是定义dao类、模型类等；
- service模块是定义service类等；
- web模块是定义controller类等；

## **SpingBoot - 入门**

### **一、简介**

> **先从 Spring 谈起**

Spring 是重量级企业开发框架 **Enterprise JavaBean（EJB）** 的替代品，Spring 为企业级 Java 开发提供了一种相对简单的方法，通过 **依赖注入** 和 **面向切面编程** ，用简单的 **Java 对象（Plain Old Java Object，POJO）** 实现了 EJB 的功能

**虽然 Spring 的组件代码是轻量级的，但它的配置却是重量级的（需要大量 XML 配置）** 。

为此，Spring 2.5 引入了基于注解的组件扫描，这消除了大量针对应用程序自身组件的显式 XML 配置。Spring 3.0 引入了基于 Java 的配置，这是一种类型安全的可重构配置方式，可以代替 XML。

尽管如此，我们依旧没能逃脱配置的魔爪。开启某些 Spring 特性时，比如事务管理和 Spring MVC，还是需要用 XML 或 Java 进行显式配置。启用第三方库时也需要显式配置，比如基于 Thymeleaf 的 Web 视图。配置 Servlet 和过滤器（比如 Spring 的`DispatcherServlet`）同样需要在 web.xml 或 Servlet 初始化代码里进行显式配置。组件扫描减少了配置量，Java 配置让它看上去简洁不少，但 Spring 还是需要不少配置。

光配置这些 XML 文件都够我们头疼的了，占用了我们大部分时间和精力。除此之外，相关库的依赖非常让人头疼，不同库之间的版本冲突也非常常见。

**不过，好消息是：Spring Boot 让这一切成为了过去。**

> **再来谈谈 Spring Boot**

**最好直白的介绍莫过于官方的介绍：**

> Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can “just run”...Most Spring Boot applications need very little Spring configuration.(Spring Boot 可以轻松创建独立的生产级基于 Spring 的应用程序,只要通过 “just run”（可能是 run ‘Application’或 java -jar 或 tomcat 或 maven 插件 run 或 shell 脚本）便可以运行项目。大部分 Spring Boot 项目只需要少量的配置即可)
**简而言之，从本质上来说，Spring Boot 就是 Spring，它做了那些没有它你自己也会去做的 Spring Bean 配置。**

 **为什么需要 Spring Boot?**

Spring Framework 旨在简化 J2EE 企业应用程序开发。Spring Boot Framework 旨在简化 Spring 开发。

![why-we-need-springboot](https://camo.githubusercontent.com/9b1c09c87e3804ad236864c1e18be529e0e7c4c8f75a44650255bef9d3b644a6/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d372f7768792d77652d6e6565642d737072696e67626f6f742e706e67)

**Spring Boot 的主要优点**

1. 开发基于 Spring 的应用程序很容易。
2. Spring Boot 项目所需的开发或工程时间明显减少，通常会提高整体生产力。
3. Spring Boot 不需要编写大量样板代码、XML 配置和注释。
4. Spring 引导应用程序可以很容易地与 Spring 生态系统集成，如 Spring JDBC、Spring ORM、Spring Data、Spring Security 等。
5. Spring Boot 遵循“固执己见的默认配置”，以减少开发工作（默认配置可以修改）。
6. Spring Boot 应用程序提供嵌入式 HTTP 服务器，如 Tomcat 和 Jetty，可以轻松地开发和测试 web 应用程序。（这点很赞！普通运行 Java 程序的方式就能运行基于 Spring Boot web 项目，省事很多）
7. Spring Boot 提供命令行接口(CLI)工具，用于开发和测试 Spring Boot 应用程序，如 Java 或 Groovy。
8. Spring Boot 提供了多种插件，可以使用内置工具(如 Maven 和 Gradle)开发和测试 Spring Boot 应用程序。

> **SpringBoot 开发环境要求**

**构建工具**

构建工具(本项目涉及的代码大部分会采用 Maven 作为包管理工具):

| **Build Tool** | **Version** |
| -------------- | ----------- |
| Maven          | 3.3+        |
| Gradle         | 4.4+        |

**开发工具推荐**

推荐使用 IDEA 进行开发。最好的 Java 后台开发编辑器，没有之一！

**Web 服务器**

Spring Boot 支持以下嵌入式 servlet 容器:

| **Name**     | **Servlet Version** |
| ------------ | ------------------- |
| Tomcat 9.0   | 4.0                 |
| Jetty 9.4    | 3.1                 |
| Undertow 2.0 | 4.0                 |

您还可以将 Spring 引导应用程序部署到任何 Servlet 3.1+兼容的 Web 容器中。

这就是你为什么可以通过直接像运行 普通 Java 项目一样运行 SpringBoot 项目。这样的确省事了很多，方便了我们进行开发，降低了学习难度。

### **二、入门配置-第一个hello world**
> **1. 新建 Spring Boot 项目**

**①你可以通过 [https://start.spring.io/](https://start.spring.io/) 这个网站来生成一个 Spring Boot 的项目。**

![start.spring.io](https://img-blog.csdnimg.cn/img_convert/69f88b648b5f94fe8fa509fc4d987057.png)

注意勾选上 Spring Web 这个模块，这是我们所必需的一个依赖。当所有选项都勾选完毕之后，点击下方的按钮 Generate 下载这个 Spring Boot 的项目。下载完成并解压之后，我们直接使用 IDEA 打开即可。

另外，阿里也有一个类似的网站 [https://start.aliyun.com/bootstrap.html](https://start.aliyun.com/bootstrap.html) ，功能甚至还要更强大点，支持生成特定应用架构的项目模板！

![](https://img-blog.csdnimg.cn/20210421220259726.png)

**②你也可以直接通过 IDEA 来生成一个 Spring Boot 的项目，具体方法和上面类似：`File->New->Project->Spring Initializr`。**

> **2. Spring Boot 项目结构分析**

成功打开项目之后，项目长下面这个样子：

![](https://img-blog.csdnimg.cn/img_convert/1f1a3ee40347e941891988cbb72ceee1.png#pic_center)

以 Application 为后缀名的 Java 类一般就是 Spring Boot 的启动类，比如本项目的启动项目就是`HelloWorldApplication` 。我们直接像运行普通 Java 程序一样运行它，由于 Spring Boot 本身就嵌入 servlet 容器的缘故，我们的 web 项目就运行成功了， 非常方便。

需要注意的一点是 **Spring Boot 的启动类是需要最外层的，不然可能导致一些类无法被正确扫描到，导致一些奇怪的问题。** 一般情况下 Spring Boot 项目结构类似下面这样

```
com
  +- example
    +- myproject
      +- Application.java
      |
      +- domain
      |  +- Customer.java
      |  +- CustomerRepository.java
      |
      +- service
      |  +- CustomerService.java
      |
      +- controller
      |  +- CustomerController.java
      |
      +- config
      |  +- swagerConfig.java
      |
```

1. `Application.java`是项目的启动类
2. domain 目录主要用于实体（Entity）与数据访问层（Repository）
3. service 层主要是业务类代码
4. controller 负责页面访问控制
5. config 目录主要放一些配置类

> **3. @SpringBootApplication 注解分析**

`HelloWorldApplication`

```java
@SpringBootApplication
public class HelloWorldApplication {
	public static void main(String[] args) {
		SpringApplication.run(HelloWorldApplication.class, args);
	}
}
```

说到 Spring Boot 启动类就不得不介绍一下 **`@SpringBootApplication` 注解**了，这个注解的相关代码如下：

```java
package org.springframework.boot.autoconfigure;
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
   ......
}
```

```java
package org.springframework.boot;
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
}
```

可以看出大概可以把 `@SpringBootApplication`看作是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合。根据 SpringBoot 官网，这三个注解的作用分别是：

- `@EnableAutoConfiguration`：启用 SpringBoot 的自动配置机制，可以根据maven依赖自动构建相关环境，比如web环境等
- `@ComponentScan`： 扫描被`@Component` (`@Service`,`@Controller`)注解的 bean，注解默认会扫描该类所在的包下所有的类，所以程序入口在最外层com.ws下。
- `@Configuration`：允许在上下文中注册额外的 bean 或导入其他配置类，被Spring扫描到的@Component都不用再加@Configuration。

> **4. 新建一个 Controller**

上面说了这么多，我们现在正式开始写 Spring Boot 版的 “Hello World” 吧。

新建一个 `controller` 文件夹，并在这个文件夹下新建一个名字叫做 `HelloWorldController` 的类。

`@RestController`是 Spring 4 之后新加的注解，如果在 Spring4 之前开发 RESTful Web 服务的话，你需要使用`@Controller` 并结合`@ResponseBody`注解，也就是说`@Controller` +`@ResponseBody`= `@RestController`。对于这两个注解，我在基础篇单独抽了一篇文章来介绍。

`com.example.helloworld.controller`

```java
@RestController
@RequestMapping("test")
public class HelloWorldController {
    @GetMapping("hello")
    public String sayHello() {
        return "Hello World";
    }
}
```

默认情况下，Spring Boot 项目会使用 8080 作为项目的端口号。如果我们修改端口号的话，非常简单，直接修改

`application.properties`配置文件即可。

`src/main/resources/application.properties`

```properties
server.port=8080
```

> **大功告成,运行项目**

运行 `HelloWorldApplication` ，运行成功控制台会打印出一些消息，不要忽略这些消息，它里面会有一些比较有用的信息。

```
2019-10-03 09:24:47.757  INFO 26326 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
```

上面是我截取的一段控制台打印出的内容，通过这段内容我们就知道了 Spring Boot 默认使用的 Tomact 内置 web 服务器，项目被运行在我们指定的 8080 端口上，并且项目根上下文路径是 "/"。

浏览器 http://localhost:8080/test/hello 如果你可以在页面正确得到 "Hello World" 的话，说明你已经成功完成了这部分内容。

> **总结**

通过本文我们学到了如何新建 Spring Boot 项目、SpringBoot 项目常见的项目结构分析、`@SpringBootApplication` 注解分析，最后实现了 Spring Boot 版的 "Hello World"。


执行main方法后，在浏览器输入请求：`http://localhost:8080/test`，会`"Hello SpringBoot"`;

### **三、SpringBoot项目打包成可运行jar**

如果希望通过`mvn package` 打包出一个可运行的jar，需要增加一个插件：`spring-boot-maven-plugin`

![QQ图片20210925170742.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210925170742-5ba1af5364c841269bceadd13cc5ceaa.png)

也可以使用插件的run功能，直接运行SpringBoot应用。

### **四、热部署**
1. 增加依赖 `spring-boot-devtools`（Debug模式下，它可以监控classpath的变化）
```
        <!-- 热部署(在debug模式下) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
```

2. 重新编译项目：手动编译（Build Project、Build Module）、自动编译（当IDEA失去焦点时，会进行自动编译，会有点延迟）

## **SpingBoot - 配置文件**

很多时候我们需要将一些常用的配置信息比如`阿里云oss配置`、`访问端口`、`context-path`、`发送短信的相关信息配置`等等放到配置文件中。
```yml
#server.port:修改端口号
server.port=8080

#设置context-path--> localhost:8080/cfg/test
server.servlet.context-path=/cfg
```

> **在之前学习中，我们在配置较简单的情况下使用了application.properties的配置文件，较复杂后我们使用application.xml的配置文件，而现在我们推出YAML类型的配置文件：**

- SpringBoot更推荐用YAML作为配置文件
- 其文件拓展名为.yml
- YAML是使用空格或TAB作为层级缩进的
- 冒号 与后面紧跟的内容之间必须要有空格或TAB
- 字符串可以不用加引号。如果有转义字符，可以使用双引号括住字符串

> **为了更快捷的完成绑定工作，可以添加依赖：`spring-boot-configuration-processor`(配置文件属性名提示)、`lombok`（在编译期间帮助生成`Getter、Setter`，需要注意的是：`Lombok`仅仅在编译期间使用即可，所以`scope`设置为`provided`即可）**


下面我们来看一下 Spring 为我们提供了哪些方式帮助我们从配置文件中读取这些配置信息。

`application.yml` 内容如下：

```yaml
wuhan2020: 2020年初武汉爆发了新型冠状病毒，疫情严重，但是，我相信一切都会过去！武汉加油！中国加油！
my-profile:
  name: 帅哥
  email: 1102733772WS@gmail.com
library:
  location: 湖北武汉加油中国加油
  books:
    - name: 天才基本法
      description: 二十二岁的林朝夕在父亲确诊阿尔茨海默病这天，得知自己暗恋多年的校园男神裴之即将出国深造的消息——对方考取的学校，恰是父亲当年为她放弃的那所。
    - name: 时间的秩序
      description: 为什么我们记得过去，而非未来？时间“流逝”意味着什么？是我们存在于时间之内，还是时间存在于我们之中？卡洛·罗韦利用诗意的文字，邀请我们思考这一亘古难题——时间的本质。
    - name: 了不起的我
      description: 如何养成一个新习惯？如何让心智变得更成熟？如何拥有高质量的关系？ 如何走出人生的艰难时刻？
```

### **一、通过 `@value` 读取比较简单的配置信息**

使用  `@Value("${property}")` 读取比较简单的配置信息：

```java
@Value("${wuhan2020}")
String wuhan2020;
```

> **application配置文件的内容可以直接通过@Value进行注入，而且可以注入数组类型，甚至map类型（key-value），但需要注意的是 `@value`这种方式是不被推荐的，Spring 比较建议的是下面几种读取配置信息的方式。**

### **二、通过`@ConfigurationProperties`读取并与 bean 绑定**

>  **`LibraryProperties` 类上加了 `@Component` 注解，我们可以像使用普通 bean 一样将其注入到类中使用。**

```java
@Component
@ConfigurationProperties(prefix = "library")
@Setter
@Getter
@ToString
class LibraryProperties {
    private String location;
    private List<Book> books;
    @Setter
    @Getter
    @ToString
    static class Book {
        String name;
        String description;
    }
}
```

这个时候你就可以像使用普通 bean 一样，将其注入到类中使用：

```java
/**
 * @author wang.shuai
 */
@SpringBootApplication
public class ReadConfigPropertiesApplication implements InitializingBean {
    private final LibraryProperties library;
    public ReadConfigPropertiesApplication(LibraryProperties library) {
        this.library = library;
    }
    public static void main(String[] args) {
        SpringApplication.run(ReadConfigPropertiesApplication.class, args);
    }
    @Override
    public void afterPropertiesSet() {
        System.out.println(library.getLocation());
        System.out.println(library.getBooks());    }
}
```

控制台输出：

```
湖北武汉加油中国加油
[LibraryProperties.Book(name=天才基本法, description........]
```

### **三、通过`@ConfigurationProperties`读取并校验**

我们先将`application.yml`修改为如下内容，明显看出这不是一个正确的 email 格式：

```yaml
my-profile:
  name: 帅哥
  email: 1102733772WS@gmail
```

> **`ProfileProperties` 类没有加 `@Component` 注解。我们在我们要使用`ProfileProperties` 的地方使用`@EnableConfigurationProperties`注册我们的配置bean：**
 ```java
/**
 * @author shuang.kou
 */
@Getter
@Setter
@ToString
@ConfigurationProperties("my-profile")
@Validated
public class ProfileProperties {
    @NotEmpty
    private String name;
    @Email
    @NotEmpty
    private String email;
  
    //配置文件中没有读取到的话就用默认值
    private Boolean handsome = Boolean.TRUE;
}
 ```

具体使用：

```java
/**
 * @author wang.shuai
 */
@SpringBootApplication
@EnableConfigurationProperties(ProfileProperties.class)
public class ReadConfigPropertiesApplication implements InitializingBean {
    private final ProfileProperties profileProperties;
    public ReadConfigPropertiesApplication(ProfileProperties profileProperties) {
        this.profileProperties = profileProperties;
    }
    public static void main(String[] args) {
        SpringApplication.run(ReadConfigPropertiesApplication.class, args);
    }
    @Override
    public void afterPropertiesSet() {
        System.out.println(profileProperties.toString());
    }
}
```

因为我们的邮箱格式不正确，所以程序运行的时候就报错，根本运行不起来，保证了数据类型的安全性：

```visual basic
Binding to target org.springframework.boot.context.properties.bind.BindException: Failed to bind properties under 'my-profile' to cn.javaguide.readconfigproperties.ProfileProperties failed:
    Property: my-profile.email
    Value: 1102733772WS@gmail
    Origin: class path resource [application.yml]:5:10
    Reason: must be a well-formed email address
```

我们把邮箱测试改为正确的之后再运行，控制台就能成功打印出读取到的信息：

```
ProfileProperties(name=帅哥, email=1102733772WS@gamil.com, handsome=true)
```

### **四、`@PropertySource`读取指定 properties 文件**

```java
@Component
@PropertySource("classpath:website.properties")
@Getter
@Setter
class WebSite {
    @Value("${url}")
    private String url;
}
```

使用：

```java
@Autowired
private WebSite webSite;
System.out.println(webSite.getUrl());//https://java.cn/
```

### **五、题外话:Spring加载配置文件的优先级**

SpringBoot默认会去加载一个应用程序配置文件名为application的文件，而且读取配置文件也是有优先级的：

![687.png](/upload/2021/09/687-07c1ab63d50a4e12926a9bde3b873b76.png)

![QQ图片20210925190635.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210925190635-272eaaeffc054bf8a17622151de846f0.png)

**而我们一般都是放在优先级最低的classjpath下，即resource/application.yml!**

更对内容请查看官方文档：https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config

## **SpingBoot - 页面模板**
SpringBoot常见的页面模板有：JSP、Freemarker、Thymeeaf（推荐）

### **Thymeleaf**
Thymeleaf是SpringBoot中推荐使用的页面模板引擎，其完全可以取代JSP，可以在没有服务器的环境下展示静态页面！

### **一、Thymeleaf - Hello World**

添加依赖后再配置文件中的默认配置
```
spring:
  thymeleaf:
    encoding: UTF-8
    prefix: classpath:/tmpates/
    suffix: .html
    enabled: true
    servlet: 
      content-type: text/html
```

**controller：**
```
@Controller
public class TestController {
    // 在html文件中用${}来获取
    @GetMapping("/helloworld")
    public String helloworld(Model model) {
        model.addAttribute("name", "Larry");
        model.addAttribute("age", 22);
        model.addAttribute("msg", "<h1>msg</h1>");
        return "01_helloworld";
    }
}
```
**html:**
```
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

This is 01_helloworld.html

<div th:text="${name}">姓名</div>
<div th:text="${age}">年龄</div>
<!--这里用th:utext可以对msg的内容进行解析-->
<p th:utext="${msg}">消息</p>

</body>
</html>
```

**总结：**
- classpath:/templates/目录下的内容只能转发访问
- 因为一些在yml的默认配置，使得返回值会加在 / 和 .html
- 在thymeleaf的控制器和html文件中，会用model.addAttribute去返回参数，而在html中用${xxx}去获取参数且th:utext会去对返回的文本解析渲染

> **thymeleaf页面模板其中包括: 注释、字面量、局部变量、三目运算符、选中变量表达式、消息表达式、链接表达式、遍历、属性设置、内置对象、视图映射等等**

### **二、Thymeleaf - 注释**

略！

### **三、Thymeleaf - 字面量**

controller:
```
@Controller
public class TestController {
    @GetMapping("/literal")
    public String literal(Model model) {
        model.addAttribute("name", "Larry");
        model.addAttribute("age", 22);
        return "03_literal";
    }
}
```

html:
```
<body>

This is 03_literal.html

<div th:text="Ilovecoding."></div>
<div th:text="${name} + ' love coding.' + ${age}"></div>
<div th:text="${name} + | love coding.|"></div>

<div th:text="'${name} love coding.'"></div>
<div th:text="|${name} love coding.|"></div>

<div th:text="${age} % 2"></div>

<hr>

<div th:text="${name} ? '存在name' : '不存在name'"></div>

<!--with定义局部变量-->
<div th:with="id=10, salary=200, nickName='SS'">
    <div th:text="'id = ' + ${id}"></div>
    <div th:text="'salary = ' + ${salary}"></div>
    <div th:text="'nickName = ' + ${nickName}"></div>
</div>

<!--th:class会做逻辑三目运算符运算-->
<div th:class="${age % 2 == 0} ? 'blue' : 'red'">
    div666
</div>

<div th:text="${age} ? ${age} : 'No Age'">

</div>

<div th:text="${age} ?: 'No Age'">

</div>
</body>
</html>
```

![QQ图片20210925194944.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210925194944-89e49326714a45a98635d2ed48a0c7fe.png)

### **四、Thymeleaf - 局部变量、三目运算**

![QQ图片20210925194948.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210925194948-fd30597b76bf457c961e541a033ef658.png)

### **五、Thymeleaf - 选中变量表达式**

![QQ图片20210925194952.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210925194952-56f3c08696c44d3cae82d94ea0410d44.png)

### **六、Thymeleaf - 消息表达式**

![QQ图片20210925194955.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210925194955-aaa91e2efe9d4936b131785cf2c7bf78.png)

### **七、Thymeleaf - 链接表达式**

![QQ图片20210925195000.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210925195000-a11cfdd894f34b95b19b767403031e99.png)

### **八、Thymeleaf - 遍历**

![QQ图片20210925195003.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210925195003-fdeab61e13694c38831de6dbe3666f63.png)

### **九、Thymeleaf - 内置对象**

> **基础对象：**
#ctx、
#vars、
#locale、
#request：HttpServletRequest、
#response：HttpServletResponse、
#session、session：HttpSession、
#servletContext、application：ServletContext

> **工具对象：**
#messages
#uris
#dates
#numbers、#strings、#objects、#bools、#arrays、#list、#sets、#maps等

### **十、视图映射**
对于视图映射，默认情况下会把`index`视图当做首页，当然了我们可以在`SpringMVCConfig`中实现`WebMvcConfigurer`，来自定义视图映射:

**SpringMVCConfig:**
```
// 修改默认配置：如首页
@Configuration
public class SpringMVCConfig implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        // 首页的映射
        registry.addViewController("/").setViewName("login");

        // 其他页面的映射
        // 请求到/comment就是映射到02_comment后拼接出来的url->{ctx}02_comment.html
        registry.addViewController("/comment").setViewName("02_comment");
//        registry.addViewController("/comment/").setViewName("02_comment");

        registry.addViewController("/i18n").setViewName("05_i18n");
//        registry.addViewController("/i18n/").setViewName("05_i18n");
    }
}
```
`registry.addViewController("/xxx").setViewName("yyy");` 当请求xxx时就会跳转到`classpath:/yyy`；也可以设置首页(请求/)，直接转发到对应页面，这样就可以省去在控制器中的转发代码！

## **SpingBoot - MyBatis**

**SpringBoot 整合 Mybatis 有两种常用的方式，一种就是我们常见的 `xml` 的方式 ，还有一种是`全注解`的方式。我觉得这两者没有谁比谁好，在 SQL 语句不太长的情况下，我觉得全注解的方式一定是比较清晰简洁的。但是，复杂的 SQL 确实不太适合和代码写在一起。**

下面就开始吧！

### **一、 开发前的准备**

> **环境参数**

- 开发工具：IDEA
- 基础工具：Maven+JDK8
- 所用技术：SpringBoot+Mybatis
- 数据库：MySQL
- SpringBoot版本：2.1.0

> **创建工程**

创建一个基本的 SpringBoot 项目，我这里就不多说这方面问题了，具体可以参考前面的文章。


> **创建数据库和 user 用户表**

我们的用户表很简单，只有 4 个字段：用户 id、姓名、年龄、余额，如下图所示：

![表信息](https://camo.githubusercontent.com/c76b0a0ecfe292bea5808f44bab06e2731c6486c7798b70365fbc09d2bbe6817/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d31312d32392f39393234383036302e6a7067)

添加了“余额money”字段是为了给大家简单的演示一下事务管理的方式。

**建表语句：**

```sql
CREATE TABLE `user` (
  `id` int(13) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `name` varchar(33) DEFAULT NULL COMMENT '姓名',
  `age` int(3) DEFAULT NULL COMMENT '年龄',
  `money` double DEFAULT NULL COMMENT '账户余额',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8
```

> **配置 pom 文件中的相关依赖**

由于要整合 springboot 和 mybatis 所以加入了artifactId 为 `mybatis-spring-boot-starter` 的依赖，由于使用了Mysql 数据库 所以加入了artifactId 为 `mysql-connector-java` 的依赖。

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.2</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

> **配置 application.properties** 

由于我使用的是比较新的Mysql连接驱动，所以配置文件可能和之前有一点不同。

```properties
server.port=8333
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/erp?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=GMT%2B8
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

注意：我们使用的 mysql-connector-java 8+ ，JDBC 连接到mysql-connector-java 6+以上的需要指定时区 `serverTimezone=GMT%2B8`。另外我们之前使用配置 Mysql数据连接是一般是这样指定`driver-class-name=com.mysql.jdbc.Driver`,但是现在不可以必须为 否则控制台下面的异常：

```error
Loading class `com.mysql.jdbc.Driver'. This is deprecated. The new driver class is `com.mysql.cj.jdbc.Driver'. The driver is automatically registered via the SPI and manual loading of the driver class is generally unnecessary.
```

上面异常的意思是：`com.mysql.jdbc.Driver` 被弃用了。新的驱动类是 `com.mysql.cj.jdbc.Driver`。驱动程序通过SPI自动注册，手动加载类通常是不必要。

如果你非要写把`com.mysql.jdbc.Driver` 改为`com.mysql.cj.jdbc.Driver `即可。

> **创建用户类 Bean**

**User.java**

```java
public class User {
    private int id;
    private String name;
    private int age;
    private double money;
    ...
    此处省略getter、setter以及 toString方法
}
```

### **二、全注解的方式**

先来看一下 全注解的方式，这种方式和后面提到的 xml 的方式的区别仅仅在于：一个将 sql 语句写在 java 代码中，一个写在 xml 配置文件中。全注方式解转换成 xml 方式仅需做一点点改变即可，我在后面会提到。

**项目结构：**

![全注解方式项目结构](https://camo.githubusercontent.com/4b80a56f03f3f136e6f664e2ee2884c29bda73fa721eb58694e87b9f784df275/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d31312d32392f313930393931302e6a7067)

> **Dao 层开发**


**UserDao.java**

```java
@Mapper
public interface UserDao {
    /**
     * 通过名字查询用户信息
     */
    @Select("SELECT * FROM user WHERE name = #{name}")
    User findUserByName(@Param("name") String name);
    /**
     * 查询所有用户信息
     */
    @Select("SELECT * FROM user")
    List<User> findAllUser();
    /**
     * 插入用户信息
     */
    @Insert("INSERT INTO user(name, age,money) VALUES(#{name}, #{age}, #{money})")
    void insertUser(@Param("name") String name, @Param("age") Integer age, @Param("money") Double money);
    /**
     * 根据 id 更新用户信息
     */
    @Update("UPDATE  user SET name = #{name},age = #{age},money= #{money} WHERE id = #{id}")
    void updateUser(@Param("name") String name, @Param("age") Integer age, @Param("money") Double money,
                    @Param("id") int id);
    /**
     * 根据 id 删除用户信息
     */
    @Delete("DELETE from user WHERE id = #{id}")
    void deleteUser(@Param("id") int id);
}
```

> **service 层**

```java
@Service
public class UserService {
    @Autowired
    private UserDao userDao;
    /**
     * 根据名字查找用户
     */
    public User selectUserByName(String name) {
        return userDao.findUserByName(name);
    }
    /**
     * 查找所有用户
     */
    public List<User> selectAllUser() {
        return userDao.findAllUser();
    }
    /**
     * 插入两个用户
     */
    public void insertService() {
        userDao.insertUser("SnailClimb", 22, 3000.0);
        userDao.insertUser("Daisy", 19, 3000.0);
    }
    /**
     * 根据id 删除用户
     */
    public void deleteService(int id) {
        userDao.deleteUser(id);
    }
    /**
     * 模拟事务。由于加上了 @Transactional注解，如果转账中途出了意外 SnailClimb 和 Daisy 的钱都不会改变。
     */
    @Transactional
    public void changemoney() {
        userDao.updateUser("SnailClimb", 22, 2000.0, 3);
        // 模拟转账过程中可能遇到的意外状况
        int temp = 1 / 0;
        userDao.updateUser("Daisy", 19, 4000.0, 4);
    }
}
```

> **Controller 层**

```java
@RestController
@RequestMapping("/user")
public class UserController {
    @Autowired
    private UserService userService;
    @RequestMapping("/query")
    public User testQuery() {
        return userService.selectUserByName("Daisy");
    }
    @RequestMapping("/insert")
    public List<User> testInsert() {
        userService.insertService();
        return userService.selectAllUser();
    }
    @RequestMapping("/changemoney")
    public List<User> testchangemoney() {
        userService.changemoney();
        return userService.selectAllUser();
    }
    @RequestMapping("/delete")
    public String testDelete() {
        userService.deleteService(3);
        return "OK";
    }
}
```

> **启动类**

```java
//此注解表示SpringBoot启动类
@SpringBootApplication
// 此注解表示动态扫描DAO接口所在包，实际上不加下面这条语句也可以找到
@MapperScan("top.snailclimb.dao")
public class MainApplication {
    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class, args);
    }
}
```

> **简单测试**

上述代码经过测试都没问题，这里贴一下根据姓名查询的测试的结果。

![根据姓名查询的测试的结果](https://camo.githubusercontent.com/9528d7db612b397853dc4c2897e7ad5bfb31a252f2d80527d56034ffa9ba2150/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d31312d32392f39323833343932302e6a7067)

### **三、xml 的方式**

**项目结构：**
![项目结构](https://camo.githubusercontent.com/edcb1349a82f8762db902a4b5ac87f818d3ccbae2f6cefe380465180329f1b02/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d31312d32392f3438353439322e6a7067)

相比于注解的方式主要有以下几点改变，非常容易实现。

> **Dao 层的改动**

我这里只演示一个根据姓名找人的方法。

**UserDao.java**

```java
@Mapper
public interface UserDao {
    /**
     * 通过名字查询用户信息
     */
    User findUserByName(String name);
}
```

**UserMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="top.snailclimb.dao.UserDao">

    <select id="findUserByName" parameterType="String" resultType="top.snailclimb.bean.User">
        SELECT * FROM user WHERE name = #{name}
    </select>
</mapper>

```

> **配置文件的改动**

配置文件中加入下面这句话：

`mybatis.mapper-locations=classpath:mapper/*.xml`


## **SpringBoot - 日志处理**
SpringBoot一般推荐**SLF4J（门面接口）** + **logback（具体实现）或者 Log4j 2** 进行日志处理！

### **一、日志的作用**

1. 在开发调试过程中，进行逻辑跟踪、查看运行结果
2. 辅助排查和定位线上问题，优化程序运行性能
3. 监控系统的运行状态、检测非授权的操作
4. 日志中蕴含了大量的用户数据，包括点击行为，兴趣偏好等

### **二、日志的发展**
最原始 -> JUL -> Log4j -> JCL -> SLF4J -> Logback -> Log4j2 

**总结：**

![QQ图片20210925202659.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210925202659-04d77796b85c404ca0ab9f8c01b950ad.png)

略！