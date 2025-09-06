## 一、项目简述

### 企业开发中常见的后台管理系统？

**CRM（客户关系管理）：**  管理客户信息，如个人信息、订单信息等；

![QQ图片20210928140559.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210928140559-5ded626505dc4242a5e30a8f297b9d87.png)

**OA（办公自动化）：** 基于工作流概念，使企业内部人员方便快捷地共享信息，高效协同工作；

![QQ图片20210928140611.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210928140611-b584b2a7e9c343e3b5feb97319cc1604.png)

**ERP（企业资源计划）：** 物资资源管理、人力资源管理、财务资源管理、信息资源管理等集成一体化；

![QQ图片20210928140615.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210928140615-e3a4ef17394348d5b4509566c6f7d8a9.png)

### 基础构建

**1. pom.xml依赖：** Spring单元测试、数据库、AOP、后端校验、接口文档Springfox、对象转换、验证码、缓存、权限控制Shiro等等。

**2. 项目框架：** 本项目前端大致使用layui min（后台管理模板），后端使用SpringBoot、Spring、SpringMVC、MyBatis/MyBatis-Plus、Freemarker、tiny-Pinyin等等。

**3. 基础概念：各种Object**

![QQ图片20210928141610.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210928141610-2a01e6a58f734987920c97ee7b3187c4.png)
![QQ图片20210928141613.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210928141613-06d261cf2613407fb30d4da557855910.png)

总结如下：

![QQ图片20210928141758.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210928141758-50bf4670b94746be8de3acbb8b225ff5.png)

> 对于数据库存储数据，可以使用到sql语句创建及生成软件： PowerDesigner；
由上面所述：传输给客户端的数据展示对象，通常对应一个页面，在有些时候，业务字段的值和最终展示的值是不一样的，比如：在数据库存储性别一般用0,1来表示，而不是直接存储男，女。

注意：在IDEA中连接数据库时，需要做如下配置：

![QQ图片20210928142504.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210928142504-c99f2bc7c7d4420b9bdb5d94efde8948.png)

**4. 项目使用到的插件工具**

> **MyBatis Generator**
- 它是MyBatis官方提供的代码生成器，可以根据数据库信息，逆向生成Model类（常用）、Mapper类、Mapper配置文件，大大节省开发者的编码时间，提高开发效率。
- 添加依赖：`mysql-connector-java`
- 只需在resources目录下准备一个generatorConfig.xml文件
![QQ图片20210928144059.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210928144059-458379fd139c4aefacd803a43d8c17bd.png)

- 配置context标签的常见内容：数据库连接、Model/XML/Mapper的生成位置（一般将生成的代码放在test中，尽量不要放在main中以防止覆盖）等等
![QQ图片20210928144102.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210928144102-0f767cf10c1c4bd69fab85107c5488c4.png)

> **MyBatis Generator存在类型转换的缺陷：只能将数据库的数据类型转换为有符号的Java数据类型（不能有效转换），所以一般使用[EasyCode](https://gitee.com/makejava/EasyCode/wikis/pages)**

> **Chrome插件：JSON-handle**

该插件可以在网页中解析成JSON对象直接展示！

> **Freemarker**

- 跟此前学过的JSP、Thymeleaf一样，Freemarker也是一款优秀的模板引擎
- 可以生成任意格式的文本文件（HTML、XML、Java等），与 web 环境无关，不依赖 web 容器
- 是在公司中非常受欢迎的一款模板引擎
- 对比JSP、Thymeleaf：JSP 只能用在web环境下，依赖于web容器；而Thymeleaf 虽然不依赖 web 容器，但它只能生成 XML 规范的文本文件，比如HTML、XML
- 添加依赖后准备一个后缀为.ftl的模板文件
![QQ图片20210928145522.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210928145522-d11d615d71af4904842a13a48fb346b4.png)

- 写生成代码以及文件存放位置：
![QQ图片20210928145525.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210928145525-be0e77e084bb409698e4fd6e0af593c9.png)

总结：
1. FreeMarker 是一款模板引擎： 即一种基于模板和要改变的数据， 并用来生成输出文本(HTML网页，电子邮件，配置文件，源代码等)的通用工具。它不是面向最终用户的，而是一个Java类库，是一款程序员可以嵌入他们所开发产品的组件。

2. 模板编写为FreeMarker Template Language (FTL)。它是简单的，专用的语言， 不是像PHP那样成熟的编程语言。 那就意味着要准备数据在真实编程语言中来显示，比如数据库查询和业务运算，之后模板显示已经准备好的数据。在模板中，你可以专注于如何展现数据， 而在模板之外可以专注于要展示什么数据。
![](http://freemarker.foofun.cn/figures/overview.png)

3. 这种方式通常被称为 MVC (模型 视图 控制器) 模式，对于动态网页来说，是一种特别流行的模式。 它帮助从开发人员(Java 程序员)中分离出网页设计师(HTML设计师)。设计师无需面对模板中的复杂逻辑， 在没有程序员来修改或重新编译代码时，也可以修改页面的样式。

4. FreeMarker 是一款模板引擎，所以说它得有一个特定的模板，再有一个对应模板里各个替换对象类型的数据（数据库里取），最后两者合并 FreeMarker 且输出成对应页面数据类型，如HTML、XML、Java等

### 项目请求步骤

由于我们不能直接让客户端访问静态资源（.html），所以：首先会使请求来到controller，controller就会调用对应的service，service会调用对应mapper，mapper就会访问数据库从而获取数据库数据后，controller才会将数据合并到Freemarker的模块里去，合并最后返回给客户端展示：
```
	list.ftl(模板) + 数据 -> list.html
```

> static可以直接被请求访问而templates不能，所以一般static放assets（CSS、image、js等静态资源），而temples放page/xx.ftlh等模板文件。

### MyBatis - Plus
MyBatis - Plus是MyBatis的增强工具，它在MyBatis的基础上只做增强不做改变，为简化开发，提交效率而生，为减小各种数据库操作（分页查询等）时减少手写sql语句！

**步骤：**
- 添加依赖 `mybatis-plus-boot-starter`
```
        <!-- 数据库 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>${druid.version}</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>${mybatis-plus.version}</version>
        </dependency>
```

- dao层Mapper接口，需要继承 BaseMapper，其中内置了基本sql语句（增删改查）
```
public interface xxxMapper extends BaseMapper<xxx> {
}
```

- service层需要继承IService
```
public interface xxxService extends IService<xxx> {

}
```

- serviceImpl具体实现
```
@Service
@Transactional
public class DictTypeServiceImpl extends ServiceImpl<DictTypeMapper, DictType> implements DictTypeService {
    @Override
    @Transactional(readOnly = true)
    public PageVo<DictTypeVo> list(DictTypePageReqVo query) {
        // 查询条件
        MpLambdaQueryWrapper<DictType> wrapper = new MpLambdaQueryWrapper<>();

        // 根据关键字查询
        wrapper.like(query.getKeyword(),
                DictType::getName,
                DictType::getValue,
                DictType::getIntro);

        // 按照id降序
        wrapper.orderByDesc(DictType::getId);

        // 分页查询
        return baseMapper
    }
}
```
通过MyBatis-Plus的增强后，mapper中不用写sql语句，而在serviceImpl里可通过 wrapper.like()获取对象属性；而在wrapper中也内置了其他功能，如排序orderByDesc()方法：按照id的降序排序等等。

### 前后端分离

**早期的前后端协作模式：**
1. 先由UI设计为Ph图片文件；
2. 在由前端生成静态页面 .html；
3. 后台才用动态模板技术（JSP、Thymeleaf、Freemarker等）生成HTML；
```
      静态页面 + 数据 -> 完整的HTML页面 -> 返回给客户端
```

存在问题：
1. 前端地位比较低，大部分工作在后台；
2. 调试，修改页面比较麻烦，需要前端后台充分配合；
3. 浪费流量（每次请求都会返回整个页面，意味着返回许多重复内容）；


**如今前后端分离：**
1. 页面保持使用静态页面即可html；
2. 静态页面（浏览器）发送异步请求（AJAX）给服务器，服务器返回Json；
3. 浏览器解析Json数据，功能生成对应的HTML标签，给用户展示；

![QQ图片20210928164710.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210928164710-08f323c1d89c4ac5a694bd5d46963dc0.png)

总结：部署两个服务器，页面服务器和Tomcat服务器；客户端首先发送请求会访问页面服务器，该服务器会返回一个静态页面 .html，当执行到需要数据时，浏览器才发送异步请求到Tomcat服务器，返回JSON数据后动态创建HTML。

### 同源策略
页面服务器与Tomcat服务器是不同两台服务器，所以有不同的端口，目前都在同一台电脑（localhost相同）所以会占用不同的端口，要想实现从一个端口请求另一个端口的Json数据（跨域），浏览器存在 **同源测试问题：**发送AJAX请求的来源于接收AJAX请求的目标必须是同源！

> **同源是指3个相同：协议、域名ip、端口**

![QQ图片20210928190147.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210928190147-d37edd86b37e4bc1bb09df551a46916a.png)

一般来说，异步请求AJAX会被同源策略影响！

**解决AJAX跨域请求的常用方法：** CORS（跨域请求共享）
1. CORS的实现需要客户端和服务器同时支持，对于客户端几乎所有的浏览器都支持，对于服务器需要返回相应的响应头，比如Access-Controll-Allow-Origin来告知浏览器这是一个允许跨域访问的请求。
2. SpringMVC实现CORS：在某个Controller上使用``@CrossOrigin`注解，允许当前Controller被跨域访问，此步骤称为局部设置
3. 在SpringMVC实现全局设置，实现WebMvcConfigurer类的addCorsMappings()方法：
```
@Configuration
public class WebCfg implements WebMvcConfigurer {
    @Autowired
    private JkProperties properties;

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        // /**表示对所有的路径开放全局跨域访问权限
        registry.addMapping("/**")
                // 开放哪些IP、端口、域名的访问权限
                .allowedOrigins("*")
                // 是否允许发送Cookie信息
                .allowCredentials(true)
                // 哪些HTTP方法允许跨域访问
                .allowedMethods("GET", "POST");
    }
}
```
如果要限制url权限，最好不要在这个配置类中写死url，所以需要将特定的url放在配置文件中application.yml中，以便随时改动。
```
jk:
  cfg:
    cors-origins:
      - http://localhost:63342
```

配置类：
```

// 扫描配置文件application.yml或关联配置文件（dev）中的以“jk”开头的配置
@ConfigurationProperties("jk")
@Component
@Data
public class JkProperties implements ApplicationContextAware {
    private Cfg cfg;
    private static JkProperties properties;

    public static JkProperties get() {
        return properties;
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        properties = this;
    }

    @Data
    public static class Cfg {
        private String[] corsOrigins;
    }
}
```
所以上述的全局配置类的url变为`"properties.getCfg().getCorsOrigins()"`：
```
@Configuration
public class WebCfg implements WebMvcConfigurer {
    @Autowired
    private JkProperties properties;

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        // /**表示对所有的路径开放全局跨域访问权限
        registry.addMapping("/**")
                // 开放哪些IP、端口、域名的访问权限
                .allowedOrigins("properties.getCfg().getCorsOrigins()")
                // 是否允许发送Cookie信息
                .allowCredentials(true)
                // 哪些HTTP方法允许跨域访问
                .allowedMethods("GET", "POST");
    }
}
```

**总结步骤：** 通过@ConfigurationProperties读取并与 bean 绑定，JkProperties类上加了 @Component 注解，我们可以像使用普通 bean 一样将其注入到WebCfg类中使用。当然也可以使用@PropertySource读取指定的整个 properties 文件

## 二、项目流程

### 项目第一步：建表
MySQL建议 - 数据类型的选择
1. 优先选择符合要求的最小数据类型
比如：非负数就使用UNSIGNED、使用INT UNSIGNED存储IP地址，而不是用字符串存储

2. 固定长度的字符串使用CHAR，比如手机号、身份证、MD5密码

3. 谨慎使用TEXT、ENUM类型
TEXT在很多时候会降低数据库的性能；当需要修改ENUM类型的待选值时，需要使用ALTER语句（需要锁表）
4. 金钱相关的数据，必须使用DECIMAL类型
5. 需要注意的是，Java中的整数类型都是有符号的，所以与MySQL类型的正确映射关系如下表所示:

![QQ图片20210929100742.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210929100742-8dc6d77316754b98b9796d5cd8f22da5.png)

**总结：** 若用常用数据类型来存储年龄，如TINYINT（占一个字节，-128 ~ 127），而当年龄超过127的时候，如果使用SMALLINT（2个字节，-32768 ~ 32767）是可以的，但十分浪费资源，最好的做法是：
```
      age TINYINT UNSIGNED   # 占用一个字节，范围为0~255
```
具体数据库建议详见 [MySQL优化规范](http://121.199.72.143:8090/archives/mysql%E4%BC%98%E5%8C%96%E8%A7%84%E8%8C%83)

### 基础配置

**1. application.yml**
```
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
  profiles:
    active: dev   #选用application-dev.yml作为选用配置yml
  jackson:
    default-property-inclusion: non_null #JSON数据只有不为空时才返回回去，以减少数据返回达到优化
  servlet:
    multipart:
      max-file-size: 20MB
      max-request-size: 20MB #上传图片设置最大大小
  resources:
    static-locations:
      - file:///${jk.upload.base-path} #静态资源的路径(文件上传)

mybatis-plus:
  type-aliases-package: com.ws.jk.pojo
  configuration:
    use-generated-keys: true
  global-config:
    db-config:
      id-type: auto
```

**2. application-dev.yml**
```
spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql://localhost:3306/new_jiakao2?serverTimezone=GMT%2B8

server:
  port: 8888

logging:
  level:
    com.ws.jk: debug

jk:
  cfg:
    cors-origins:
      - http://localhost:63342
  upload:      #文件上传路径
    base-path: F:/new_jk/
    upload-path: upload/
    image-path: img/
    video-path: video/
```

**3. CORS：略**

**4. 分页查询的MyBatis-Plus配置**

```
// 分页查询的MyBatis-Plus配置
@Configuration
@MapperScan("com.ws.jk.mapper")
public class MyBatisPlusCfg implements InitializingBean {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        PaginationInnerInterceptor innerInterceptor = new PaginationInnerInterceptor(DbType.MYSQL);
        // 当页数超过总页数时，自动跳回到第1页
        innerInterceptor.setOverflow(true);
        interceptor.addInnerInterceptor(innerInterceptor);
        return interceptor;
    }

    @Bean
    public ConfigurationCustomizer configurationCustomizer() {
        return configuration -> configuration.setUseDeprecatedExecutor(false);
    }
```

### 思考：如何保证数据一致性？
目前公司一般是不要求用外键来保证数据一致性问题：如（重庆是渝，下面还有各种区县）。当在删除重庆这个省份时，如果在城市里还有其下的几个区县信息，该如何处置？
之前由于外键的关联是无需处理的，而现在没有外键就需要在应用层来进行约束：到底是连省份下面的城市一起删除，还是在没有城市后才能删除省份呢？

**如何保证数据一致性？**

1. 方法一：在service的具体实现类Impl中重写父类的remove()方法，在删除之前先检查数据是否在其他表被关联！
而方法一存在问题：十分麻烦！ remove开头的方法有很多，MyBatis内置的如removeById()和removeByIds()等等；所以导致需要处理的表也可能很多；更新操作也需要处理，不能传入没有省份的城市信息，防止恶意传入数据，占用资源。

2. 方法二：编写保证一致性的框架（注解） 在PO中使用@ForeignField设置哪个属性是有关联外键，在扫描整个PO后就知道表与表之间的联系（注解信息）：
```
    @ForeignField(DictType.class) // 外键：引用DictType 默认引用DictType的id主键
```
采用Spring AOP，只要调用了remove开头的方法（removeById()和removeByIds()）就会先进行检查

```
// Spring AOP面向切面编程：拦截指定路径下remove开头的方法，以满足数据一致性
@Aspect
@Component // 有此注解才能让Spring Boot初始化
public class ForeignAspect implements InitializingBean {
    private static final String FOREIGN_SCAN = "classpath*:com/ws/jk/pojo/po/**/*.class";
    @Autowired
    private ApplicationContext ctx; // spring容器
    @Autowired
    private ResourceLoader resourceLoader;

    //Around里面放切入点表达式 @Around(返回值 具体方法的路径)
    @Around("execution(* com.ws.jk.service..*.remove*(..))")
    public Object handleRemove(ProceedingJoinPoint point) throws Throwable {
        Object target = point.getTarget(); // 获取是哪个类型对象调用remove开头的方法（后面需要(IService<?>)强制转换）
        if (!(target instanceof IService)) return point.proceed();

        // 获取具体的是哪个Service对象（返回com.ws.jk.pojo.po.DictType这种类型的类名）
        Class<?> poCls = ((IService<?>) target).getEntityClass();

        // 传入类名，返回该类的表格信息
        ForeignTableInfo table = ForeignTableInfo.get(poCls);
        if (table == null) return point.proceed();

        // 主键
        ForeignFieldInfo mainField = table.getMainField();
        if (mainField == null) return point.proceed();

        // 获取外键约束
        List<ForeignFieldInfo> subFields = mainField.getSubFields();
        if (CollectionUtils.isEmpty(subFields)) return point.proceed();

        // 获取参数值
        Object arg = point.getArgs()[0];
        List<Object> ids;

        // 以下逻辑判断：无论是一个id还是多个id都会放到List这个集合里去
        if (arg instanceof List) {
            ids = (List<Object>) arg;
        } else {
            ids = new ArrayList<>();
            ids.add(arg);
        }

        for (ForeignFieldInfo subField : subFields) {
            ForeignTableInfo subTable = subField.getTable();
            BaseMapper<Class<?>> mapper = getMapper(subTable.getCls());
            QueryWrapper<Class<?>> wrapper = new QueryWrapper<>();
            wrapper.in(subField.getColumn(), ids);
            if (mainField.getCascade() == ForeignCascade.DEFAULT) { // 默认
                Integer count = mapper.selectCount(wrapper);
                if (count == 0) continue;
                JsonVos.raise(String.format("有%d条【%s】数据相关联，无法删除！", count, subTable.getTable()));
            } else { // 删除关联数据
                mapper.delete(wrapper);
            }
        }
        return point.proceed();
    }
}
```

**步骤：** 先在foreign包中ForeignAspect这个AOP类中，切入具体是哪种方法后`@Around("execution(* com.ws.jk.service..*.remove*(..))")`，获取哪个对象调用了以remove开头的方法`getTarget()`，再由之前注解扫描到的表与表之间的关联（逻辑处理）。

> **注解 + AOP => 数据一致性**

### 二级联动
当选择省份时会在后一个下拉框出现该省的城市，可以在点击选择一个省份后在发送请求加载相应的城市，这种适合于数据量较大的场合；而省份城市数据较小时，可以一次全加载。


### 后端校验

![QQ图片20210929160249.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210929160249-0be845252f18407b9fa4a8be29f1a0cf.png)

由上图可知，在特殊数据存储操作中，可能数据库对数据有要求如不能为空、数据格式不对等等，上图页面只是在前端进行了限制，保证了不满足要求的数据是不能存储到数据库中的。
而真实开发却不仅如此，当不法分子知道了save、remove等uri之后，可能会绕过前端发送数据请求：Postman!
该工具可以在不通过前端校正的情况下，直接向服务器发送数据请求，并成功保存数据，极大的降低了数据库存储数据的安全性。**所以必须进行后端验证**。

1. 方法一（不推荐）：在相应的请求的save()方法中进行逻辑判断，如不能为空等等。但是此方法会将业务代码与校验代码耦合！

2. 方法二（推荐）：hibernate-validator框架，一直在Java中常用的一款后端校验的框架。

> **方法二的Model参数校验使用步骤：**
```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

```

- 在Model的getter或成员变量上加相关的校验注解@NotBlank()；
```
// 用来做请求的Vo（保存相关）
@Data
public class DictTypeReqVo {
    private Integer id;

    @NotBlank(message = "名称不能为空")
    private String name;

    @NotBlank(message = "值不能为空")
    private String value;

    private String intro;
}
```
- 在Model参数上加@Valid注解；

```
@Validated
public abstract class BaseController<Po, ReqVo> {
    protected abstract IService<Po> getService();
    protected abstract Function<ReqVo, Po> getFunction(); // reqVo -> Po

    // @Valid后端校验
    @PostMapping("/save")
    @ApiOperation("添加或更新")
    public JsonVo save(@Valid ReqVo reqVo) {
        // 这里的getFunction()方法需要在每个子类的具体类型中分别实现（因为MapStructs不支持泛型）
        Po po = getFunction().apply(reqVo); // reqVo -> Po (apply()方法应用一下)
        if (getService().saveOrUpdate(po)) {
            return JsonVos.ok(CodeMsg.SAVE_OK);
        } else {
            return JsonVos.raise(CodeMsg.SAVE_ERROR);
        }
    }
}
```

- 校验失败时，会抛出异常 `org.springframework.vaildation.BindException`，这样便可以通过`BindException.getBindingResult().getAllErrors()拿到所有的错误信息

```
    private JsonVo handle(BindException be) {
        List<ObjectError> errors = be.getBindingResult().getAllErrors();
        // 函数式编程的方式：stream(map方法会遍历数组errors里的每个对象,每次都会执行getDefaultMessage并从其返回值获取信息并放到新的List中)
        List<String> defaultMsgs = Streams.map(errors, ObjectError::getDefaultMessage);
        String msg = StringUtils.collectionToDelimitedString(defaultMsgs, ", "); // 将数组存放的错误信息以","进行拼接返回
        return JsonVos.error(msg);
    }
```

> **方法二的非Model参数校验使用步骤：**

- 在Controller上加@Validated注解；在非Model参数上加相关的校验注解；
```
@Validated
public abstract class BaseController<Po, ReqVo> {
    protected abstract IService<Po> getService();
    protected abstract Function<ReqVo, Po> getFunction(); // reqVo -> Po

    // 公共的方法:remove\save
    // @RequestParam 在swagger中告诉这是一个普通的参数（query)   @RequestBody为Body参数
    // 这是返回JsonVo就可以了,其属性有code和msg
    @PostMapping("/remove")
    @ApiOperation("删除一条或多条数据")
    public JsonVo remove(@NotBlank(message = "id不能为空") @RequestParam String id) {
        if (getService().removeByIds(Arrays.asList(id.split(",")))) {
            return JsonVos.ok(CodeMsg.REMOVE_OK);
        } else {
            return JsonVos.raise(CodeMsg.REMOVE_ERROR);
        }
    }
}
```
- 校验失败时，会抛出异常 `javax.validation.ConstraintViolationException`，这样便可以通过`ConstraintViolationException.getConstraintViolations()`拿到所有的错误信息
```
    // 处理后端校验ConstraintViolationException抛出的异常(非model参数校验)
    private JsonVo handle(ConstraintViolationException cve) {
        List<String> msgs = Streams.map(cve.getConstraintViolations(), ConstraintViolation::getMessage);
        String msg = StringUtils.collectionToDelimitedString(msgs, ", ");
        return JsonVos.error(msg);
    }
```

> **后端校验的常用注解**

|   名称    | 解释                     |
|---------- |--------------------------|
|@NotNull   | 不能为null，但可以为empty  |
|@NotEmpty  | 不能为null，而且长度必须大于0 |
|@NotBlank  | 只能作用在String上，不能为null，且去除空格后长度必须大于0 |
|@AssertFalse | 必须为false |
|@AssertTrue | 必须为false |
|@Max、@DecimalMax| 必须为一个不大于指定值的数字 |
|@min、@DecimalMin| 必须为一个不小于指定值的数字 |
|@Range  | 必须在一个合适的范围 | 
|@Pattern | 必须符合指定的正则表达式 |
等等

> **自定义校验**
```
// 自己编写的Bool后端校验："只能是0和1" （注解类）：用在启用/禁止功能键上
@Documented // 生成java文档
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD) // 使用范围：属性
@Constraint(validatedBy = BoolNumber.BoolNumberValidator.class)
public @interface BoolNumber {
    String message() default "只能是0和1"; // 传入自定义文字信息

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

    // 具体实现校验
    class BoolNumberValidator implements ConstraintValidator<BoolNumber, Short> {
        @Override
        public boolean isValid(Short num, ConstraintValidatorContext constraintValidatorContext) {
            return num == null || num == 0 || num == 1;
        }
    }
}
```
```
    @BoolNumber(message = "是否禁用只能是0和1")
    @ApiModelProperty("是否禁用【0代表不禁用（启用），1代表禁用，默认0】")
    private Short disabled;
```

> **快速失败**： 默认情况下是检测完所有的错误后再统一抛出异常，也可以设置快速失败：只要检测到某一个特定错误，就直接抛出异常，不再往下检查。

![QQ图片20210929204204.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210929204204-58ae5e9e7e7d4c1d878f55a7ce186376.png)


### Swagger
**何为 Swagger ？** 简单来说，Swagger 就是一套基于 OpenAPI 规范构建的开源工具，可以帮助我们设计、构建、记录以及使用 Rest API。

**为何要用 Swagger ?** 前后端分离的情况下，一份 Rest API 文档将会极大的提高我们的工作效率。前端小伙伴只需要对照着 Rest API 文档就可以搞清楚一个接口需要的参数以及返回值。通过 Swagger 我们只需要少量注解即可生成一份自带 UI 界面的 Rest API 文档，不需要我们后端手动编写。并且，通过 UI 界面，我们还可以直接对相应的 API 进行调试,省去了准备复杂的调用参数的过程。

详见[Swagger详解](http://121.199.72.143:8090/archives/springboot20ji-cheng-swagger-guan-fang-starterknife4jzeng-qiang-fang-an)

### 三级联动
> **在examPlaceCourse下的list/save.html看三级联动！ 具体略**

### 对POJO问题项目重构

> 提出问题：d当进行添加数据时，在Swagger中会有解释性注解语句，比如不能为空，不可为负数等等。而在请求数据库返回数据时，这些文字显得十分冗余，所以要用到之前提到的VO、PO、DTO等？ 所以进行项目重构：

![QQ图片20210930125126.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720210930125126-f9db8701b1974fe38bc097bc811f7ab7.png)


> 重构之前的流程

![QQ图片20210930125238.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720210930125238-4cc502dae7284afcad15dceb849c2f55.png)

**存在问题：** 若controller又有一个方法login也是用userPo，但它不完全使用到userPo的全部属性，所以会将其中的一些方法进行隐藏，而save方法又不需要对其任何参数进行隐藏，因为save方法用来生成在Swagger文件中。

![QQ图片20210930125705.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720210930125705-4c5f8aa31f284a29abaf134e23293d29.png)

以上是在查询得到返回值，可以看出返回值均为userPo，导致了所有请求都是用一个Po对象。当我们在一个请求设置了Po对象的属性，比如隐藏、注解等等，其操作都会影响其他请求方法：**一个Po走遍全天下！**

> 重构之后的流程

![QQ图片20210930130111.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720210930130111-0d47045302c040f4b64623fddc840542.png)

当客户端传入请求时是Vo对象，可以由于项目太复杂会将Vo对象转成DTO后再交给service处理（本项目没有用到DTO），所以直接将Vo转成Po给service层（转成Po是由于我们使用到了MyBatis-Plus的save方法），而当MyBatis-Plus的内置方法不能满足要求时，可以自己重写，这时就可以直接传入Vo对象，**而最后的数据库访问的mapper的对象肯定是Po对象！**

![QQ图片20210930130116.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720210930130116-c1a001e6bcf5497b8582efe2169a4f00.png)

以上是list的返回值：mapper.select查询数据库是默认返回Po对象，而controller是返回Vo对象（返回值），而传入的是userListReqVo（请求参数）！

> 在项目中存在在数据库查询出来的Po对象，需要转换为Vo对象向外展示：**对象转换MapStructs插件:**

```
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct</artifactId>
            <version>1.4.1.Final</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct-processor</artifactId>
            <version>1.4.1.Final</version>
            <scope>provided</scope>
        </dependency>
```

**先生成接口，并声明要转换的对象**

```
/**
 * 数据类型转换接口
 * ReqVo -> Po ：客户端返回的Vo转Po
 * Po -> Vo ：从数据库查询到的Po转Vo
 */
@Mapper(uses = {
        MapStructFormatter.class
})
public interface MapStructs {
    // 接口里写的变量（INSTANCE）都是 public static
    // 根据Mappers.getMapper()生成代理对象的实例INSTANCE，后面直接INSTANCE::po2vo调用下面的方法
    MapStructs INSTANCE = Mappers.getMapper(MapStructs.class);

    // Po -> Vo ：从数据库查询到的Po转Vo
    DictItemVo po2vo(DictItem po); // 先声明：传入一个DictItem的po返回一个DictItemVo
    DictTypeVo po2vo(DictType po);
    ExamPlaceVo po2vo(ExamPlace po);
    PlateRegionVo po2vo(PlateRegion po);
    ExamPlaceCourseVo po2vo(ExamPlaceCourse po);

    // 这是由于SysUser的po和vo的LoginTime两者数据类型不一样，所以需要手动(通过注解来找对应方法)
    @Mapping(source = "loginTime",
            target = "loginTime",
            qualifiedBy = MapStructFormatter.Date2Millis.class)
    SysUserVo po2vo(SysUser po);
    LoginVo po2loginVo(SysUser po);
    SysRoleVo po2vo(SysRole po);
    SysResourceVo po2vo(SysResource po);

    // ReqVo -> Po ：客户端返回的Vo转Po
    DictItem reqVo2po(DictItemReqVo reqVo);
    DictType reqVo2po(DictTypeReqVo reqVo);
    ExamPlace reqVo2po(ExamPlaceReqVo reqVo);
    PlateRegion reqVo2po(PlateRegionReqVo reqVo);
    ExamPlaceCourse reqVo2po(ExamPlaceCourseReqVo reqVo);
    SysUser reqVo2po(SysUserReqVo reqVo);
    SysRole reqVo2po(SysRoleReqVo reqVo);
    SysResource reqVo2po(SysResourceReqVo reqVo);
}

```

**实例**

```
    // 这里是返回所有数据：PageJsonVo
    @GetMapping("/list")
    @ApiOperation("查询所有")
    @RequiresPermissions(Constants.Permisson.DICT_TYPE_LIST)
    public DataJsonVo<List<DictTypeVo>> list() {
        // Streams.map()表示遍历service.list()查出来的每一个Po对象通过MapStructs.INSTANCE::po2vo函数转换为对应Vo对象
        // 这里前后代码都集成到了Streams工具类中
        // ok(): 由PageVo转换为PageJsonVo
        return JsonVos.ok(Streams.map(service.list(), MapStructs.INSTANCE::po2vo));
    }
```

### MyBatis-Puls的自动分页功能
在ExamPlaceCourseMapper中sql语句会传入分页的参数，当MyBatis-Plus知道你传入了page分页参数，会自动在sql语句自动生成LIMIT代码，十分智能！

**ExamPlaceCourseMapper**
```
<mapper namespace="com.ws.jk.mapper.ExamPlaceCourseMapper">
    <resultMap id="rmSelectPageVos"
               type="ExamPlaceCourseVo">
        <!--这里属性名和列名一一对应，可以不用写-->
        <id property="id" column="id"/>
        <result property="name" column="name"/>
        <result property="price" column="price"/>
        <result property="type" column="type"/>
        <result property="placeId" column="place_id"/>
        <result property="provinceId" column="province_id"/>
        <result property="cityId" column="city_id"/>
        <result property="cover" column="cover"/>
    </resultMap>

    <!--根据课程查考场，考场对应有province_id/city_id,就可以找到对应省份和城市-->
    <select id="selectPageVos"
            resultMap="rmSelectPageVos">
        SELECT
            c.id,
            c.name,
            c.price,
            c.type,
            c.intro,
            c.place_id,
            c.cover,
            p.province_id,
            p.city_id
        FROM exam_place_course c
            JOIN exam_place p ON p.id = c.place_id
        ${ew.customSqlSegment}       <!--分页-->
    </select>
</mapper>
```

**根据查询条件，自动生成查询语句**
```
// 分页查询的MyBatis-Plus配置
@Configuration
@MapperScan("com.ws.jk.mapper")
public class MyBatisPlusCfg implements InitializingBean {
    @Override
    public void afterPropertiesSet() throws Exception {
        /*
        首先，拥有lambda cache的实体类Entity，才能使用LambdaQueryWrapper<Entity>，
        默认情况下，只有BaseMapper<Entity>中的Entity类，才拥有lambda cache,
        其他类需要通过TableInfoHelper手动添加lambda cache
        */
        MapperBuilderAssistant assistant = new MapperBuilderAssistant(new MybatisConfiguration(), "");
        TableInfoHelper.initTableInfo(assistant, ExamPlaceCourseVo.class);
    }
}

```

### 权限管理（重点）

权限管理：对已经 **认证成功** 的用户进行 **授权！**
- **认证成功：** 比如使用用户名、密码登录成功
- **授权：** 比如赋值相应的操作权限（资源）

> **实现方案：**

建议采用RBAC（Role-based Access Control），以角色为基础的访问控制：
- 用户跟角色挂钩、角色与资源挂钩，设计成5张表
- 用户表（sys_user）、角色表（sys_role）、资源（sys_resource）、用户角色表（sys_user_role）、角色资源表（sys_role_resource）

![QQ图片20211004152639.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211004152639-6a9d6457fa6e4af99ed051e8fd1517a0.png)

**由上可知：添加一个角色，一个人对应一个角色，而各种角色只对应一类权限资源，这样就不用一个人直接对应各种资源了！**

> 点一：save与update

![QQ图片20211004153148.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211004153148-7e22bd1527ba4ce7a2d3bd8e4c362547.png)

在前端页面，sys/user中有save和update.html两个页面，这是因为由上图，save时是必须要输入密码的，而编辑更新时是不用在密码处设置的，不设置就表示默认使用原来的密码！

> 点二：在项目中，要记录最后一次登录时间：

**在数据库存储Po对象中，用Date定义**
```
    /**
     * 最后一次登录的时间
     */
    private Date loginTime;
```

**展示页面的Vo对象，用Long定义**

```
    @ApiModelProperty("最后一次登录的时间")
    // 一般返回UNIX时间戳(校正时间),所以不是Data而是Long(即从1970年1月1日到现在的毫秒数，最后前端拿到毫秒数再转换为自己想要的日期格式)
    private Long loginTime;
```

由于在客户端需要展示最后一次登录的时间，以下在Impl会进行分页查询，最后会返回由Po转Vo对象！
```
    // 用户左上角查询条件（分页查询）
    @Override
    @Transactional(readOnly = true)
    public PageVo<SysUserVo> list(SysUserPageReqVo reqVo) {
        MpLambdaQueryWrapper<SysUser> wrapper = new MpLambdaQueryWrapper<>();
        wrapper.like(reqVo.getKeyword(), SysUser::getNickname, SysUser::getUsername);
        wrapper.orderByDesc(SysUser::getId);
        return baseMapper
                .selectPage(new MpPage<>(reqVo), wrapper)
                .buildVo(MapStructs.INSTANCE::po2vo);
    }
```

由于框架无法自动实现Date与Long之间的转换，所以需要自定义实现两种类型之间的转换：

**MapStructs**
```
    // 这是由于SysUser的po和vo的LoginTime两者数据类型不一样，所以需要手动(通过注解来找对应方法)
    @Mapping(source = "loginTime",
            target = "loginTime",
            qualifiedBy = MapStructFormatter.Date2Millis.class)
    SysUserVo po2vo(SysUser po);
    LoginVo po2loginVo(SysUser po);
    SysRoleVo po2vo(SysRole po);
    SysResourceVo po2vo(SysResource po);
```

**MapStructFormatter**
```
public class MapStructFormatter {
    // Date -> Long(millis):通过注解来找对应方法
    @Qualifier // 固定格式
    @Target(ElementType.METHOD)
    @Retention(RetentionPolicy.CLASS)
    public @interface Date2Millis {}

    // 具体实现转换
    @Date2Millis
    public static Long date2millis(Date date) {
        if (date == null) return null;
        return date.getTime();
    }


    // Long(millis) -> Date
    @Qualifier
    @Target(ElementType.METHOD)
    @Retention(RetentionPolicy.CLASS)
    public @interface Mills2Date {}

    @Mills2Date
    public static Date millis2date(Long mills) {
        if (mills == null) return null;
        return new Date(mills);
    }
}

```

> 点三：在前端代码中的js/common.js的Tree和Transfer是分别对系统模块中的角色资源的树状图组件的封装，以及对用户的角色框图进行封装

![QQ图片20211004154754.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211004154754-42ee7989209e4b95aaecc25aec300416.png)

由上图逻辑可知：在角色里的数据，除了有显示出来的name，还有id信息，更改后只需发送给服务器的只是id值。

> 在进行角色的选择操作时，在编辑操作时，可能是存在之间就拥有的一些角色信息，即我们当前的操作可能是添加角色、删除角色或者换掉角色，所以我们是无法使用saveOrUpdate来完成的： **做法就是：在编辑更新操作时，我们会先删掉之前所有角色，待操作完成后，直接再保存！**

**权限管理：** 当用户未登录或者用户的角色权限限制，导致一些资源是不可以被访问的，需要对其请求进行拦截

以前的操作：

![QQ图片20211004161626.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211004161626-7451d1c14bea4f0193087bb3c0531738.png)

1. 当客户端发送用户密码等登录请求后，如果用户名、密码正确，服务器会创建一个跟客户端相关的Session（其中包括Session的id以及user信息）；
2. 将Session的id以Cookie的形式返回给客户端（Set-Cookie），且客户端Cookie：1234的id信息；
3. 下次当客户端发送访问该用户模块的请求，会带上Cookie在Filter中进行拦截验证，看是否存在Cookie的id为1234的信息；
4. 成功则将该用户模块信息返回给客户端；

流程总结：
![QQ图片20211004162329.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211004162329-84f07bcf38914acb80ffc31a42893388.png)

> Cookie和Session是浏览器特有的属性，移动端默认是没有这些的！

> **简单分布式架构**

![QQ图片20211004163047.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211004163047-69b91136ec7341dcbefab1f342d28fb2.png)

redis是分布式缓存，放在内存中，查询数据库数据前会在redis缓存中查询看有没有想要的数据，如果没有才会去访问数据库，同理Session应放在分布式缓存中。

> **客户端身份验证 - 基于token**

![QQ图片20211004163438.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211004163438-2451c4db1f5049a299ee0685207edddb.png)

**和上面Cookie、Session流程是差不多的，就是服务器返回token后需要客户端手动存储到本地，前端、ios、android都有自己的存储形式；**

![QQ图片20211004163507.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211004163507-5ad04f8c52c24b278cc1b810a03b6fac.png)

注意：若使用分布式架构，将token直接放到分布式缓存redis中。

一般的企业的方案：token + redis：

> **token**
对于token这种非业务请求，放在请求头中，当然了token会存在在每一个请求中，即所有的请求都要检查token，但是还是有特殊情况：**登录请求**

登录是新用户时，本身是没有token的，所以需要一个tokenFilter来拦截：哪些uri需要token，哪些不需要！

### Shiro框架实现权限管理
Shiro是Apache推出的安全管理框架，比SpringSecurity更加简单易用，其有2大核心功能：认证（比如登录认证，只有合法用户才能登录进入系统），授权（比如设置每个合法用户的权限范围，即可以对哪些资源进行哪些操作）

> **核心类型：SecurityManager（安全管理器）、Subject（需要进行认证和授权的主体，如用户）、Authenticator（认证器）、Authorizer（授权器）、Realm（相当于数据源，可以用于获取主体的权限信息）**

![QQ图片20211004164654.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211004164654-d0d1715bfb094162b200b2ea16a817c5.png)

**流程总结如下：**
```
/*
自定义流程:
认证流程：
1、Subject.login(token): 登录请求
2、SecurityManager -> Authenticator(认证器) -> Realm【AuthorizingRealm】(信息数据源)
3、info = AuthorizingRealm.doGetAuthenticationInfo(token): 根据token信息查询对应的用户信息（比如去数据库中查找）,并返回info(必有值)
4、CredentialsMatcher.doCredentialsMatch(token, info): 判断token、info的Credentials是否匹配
(CredentialsMatcher：专门用来判断Credentials是否正确)

判断权限、角色流程：
1、Subject.isPermitted(permission)、Subject.hasRole(role)等
2、SecurityManager -> Authorizer -> Realm【AuthorizingRealm】
3、info = AuthorizingRealm.doGetAuthorizationInfo(principal的集合): 根据principal查询对应的角色、权限信息
4、根据返回的info信息判断权限、角色是否正确
 */
```

> **Shior有内置Filter，只需要设置如何拦截即可！**

![QQ图片20211004165647.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211004165647-eb80908e3527420799893393f60f0539.png)

anon：直接放行（验证码、登录、Swagger、全局Filter异常处理、上传内容等等均可直接放行），其他的请求就必须要通过在Shiro配置类中自定义的Filter：
```
@Configuration
public class ShiroCfg {
    // 注入自定义的Realm:TokenRealm
    @Bean
    public Realm realm() {
        return new TokenRealm(new TokenMatcher());
    }

    /**
     * ShiroFilterFactoryBean用来告诉Shiro如何进行拦截
     * 1.拦截哪些URL
     * 2.每个URL需要进行哪些filter
     */
    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(Realm realm, JkProperties properties) {
        ShiroFilterFactoryBean filterBean = new ShiroFilterFactoryBean();

        // 安全管理器
        filterBean.setSecurityManager(new DefaultWebSecurityManager(realm));

        // 添加一些自定义Filter
        Map<String, Filter> filters = new HashMap<>();
        filters.put("token", new TokenFilter());
        filterBean.setFilters(filters);

        // 设置URL如何拦截(**:多层; anon：匿名访问，前面的url直接放行; token: 自定义Filter; 存放url必须用LinkedHashMap)
        Map<String, String> urlMap = new LinkedHashMap<>();
        // 验证码
        urlMap.put("/sysUsers/captcha", "anon");
        // 用户登录
        urlMap.put("/sysUsers/login", "anon");
        // swagger(swagger有两个访问url; swagger内部存在的一个需要访问的url)
        urlMap.put("/swagger*/**", "anon");
        urlMap.put("/v2/api-docs/**", "anon");
        // 全局Filter的异常处理: ERROR_URI = "/handleError"
        urlMap.put(ErrorFilter.ERROR_URI, "anon");
        // 上传的内容(/upload/**)
        urlMap.put("/" + properties.getUpload().getUploadPath() + "**", "anon");
        // 其他请求都要经过我自定义的Filter，所以填token（自定义Filter的key）
        urlMap.put("/**", "token");
        filterBean.setFilterChainDefinitionMap(urlMap);

        return filterBean;
    }

    /**
     * 解决：在各个controller的请求上加@RequiresPermissions导致控制器接口404问题
     */
    @Bean
    public DefaultAdvisorAutoProxyCreator proxyCreator() {
        DefaultAdvisorAutoProxyCreator proxyCreator = new DefaultAdvisorAutoProxyCreator();
        proxyCreator.setUsePrefix(true);
        return proxyCreator;
    }
}
```
注意：在上面代码中，用LinkedHashMap<>()来顺序存储添加的URL，为什么？

![QQ图片20211004170234.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211004170234-deecfbf19104430a818ec4b68935e87f.png)

上图可以知道：HashMap与LinkedHashMap都存放上面代码的put进去的url（hashMap是无序的，存在上面图中这种存储情况）

1. 当正常情况下，请求/sysUsers/captcha是符合LinkedHashMap的第一条，“anon”直接放行；
2. 而使用了hashMap存储，则“/**”也是匹配成功的，“token”表示必须经过自定义Filter，是错误的！
3. 所以必须使用有序的LinkedHashMap！
4. 当使用了多个自定义的filter，只需用逗号隔开即可：
```
	urlMap.put("/**","token1","token2")
```

### 异常处理
之前我们的全局异常会统一处理，但该异常只会抛出来自controller的异常，而现在我们加入了shior权限管理，有些请求根本就没有到达controller，在filter（TokenFilter）就被拦截了，所以这种异常是无法返回到客户端的！

**解决方案：** 只需在所有Filter之前添加一个ErrorFilter处理后面的tokenFliter抛出的异常，当ErrorFilter拿到由tokenFliter的异常信息之后，转发到ErrorController并在这里直接抛出异常，这样操作对其异常进行返回客户端，这样所有的异常均是被controller抛出的，因此所有Controller中异常会被统一处理！

**ErrorFilter：转发到ContextPath/handleError**
```
public class ErrorFilter implements Filter {
    public static final String ERROR_URI = "/handleError";

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        try {
            chain.doFilter(request, response); // 进入下一个链式调用（比如Filter、拦截器、控制器等）
        } catch (Exception e) {
            // ErrorFilter(Filter1)后面的所有Filter抛出异常都会在这里被拦截，转发给ErrorController
            request.setAttribute(ERROR_URI, e);
            request.getRequestDispatcher(ERROR_URI).forward(request, response);
        }
    }
}
```

**ErrorController：拦截此异常，直接抛出**
```
@RestController
public class ErrorController {
    @RequestMapping(ErrorFilter.ERROR_URI)
    public void handle(HttpServletRequest request) throws Exception {
        // 取出并直接抛出异常
        throw (Exception) request.getAttribute(ErrorFilter.ERROR_URI);
    }
}
```

![QQ图片20211004193418.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211004193418-82f4480c6e0d49798ca6e2af435ba9f9.png)

> **多Filter场合**

![QQ图片20211004193538.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211004193538-e5f59400ee8240629d9c3678c4b0b1a6.png)

当存在多个Filter时，从客户端的请求过来，经过Filter1会调用chain.doFilter才会到Filter2，同理直到进行controller的具体业务请求（如login），所以将ErrorFilter设置为Filter1，通过try-catch便可以拦截到后面所有Filter的异常，后面就直接转发到ErrorController并取出和抛出异常，这样的异常才会去CommonExceptionHander作统一处理！

```
// 通用异常处理
@RestControllerAdvice // 将return后面的对象直接返回，而不是像ControllerAdvice那样返回路径，拼接后找到对应jsp
@Slf4j
public class CommonExceptionHandler {
    @ExceptionHandler(Throwable.class) // 拦截异常的范围,Throwable.class:全部异常拦截
    @ResponseStatus(code = HttpStatus.BAD_REQUEST) // 等价于response.setStatus(400);
    public JsonVo handle(Throwable t) {
        log.error("handle", t);

        // 一些可以直接处理的异常
        if (t instanceof CommonException) {
            return handle((CommonException) t);
        } else if (t instanceof BindException) { // BindException是后端校验hibernate-validator抛出的
            return handle((BindException) t);
        } else if (t instanceof ConstraintViolationException) {
            return handle((ConstraintViolationException) t);
        } else if (t instanceof AuthorizationException) {
            return JsonVos.error(CodeMsg.NO_PERMISSION);
        }

        // 处理cause异常（导致产生t的异常）
        Throwable cause = t.getCause();
        if (cause != null) {
            return handle(cause);
        }

        // 其他不知道的异常：默认返回CodeMsg=400（没有cause的异常）：可能是系统异常
        return JsonVos.error();
    }

    // 处理普通异常，直接返回Code和Message
    private JsonVo handle(CommonException ce) {
        return JsonVos.error(ce.getCode(), ce.getMessage());
    }

    // 处理后端校验BindException抛出的异常(model参数校验)
    private JsonVo handle(BindException be) {
        List<ObjectError> errors = be.getBindingResult().getAllErrors();
        // 函数式编程的方式：stream(map方法会遍历数组errors里的每个对象,每次都会执行getDefaultMessage并从其返回值获取信息并放到新的List中)
        List<String> defaultMsgs = Streams.map(errors, ObjectError::getDefaultMessage);
        String msg = StringUtils.collectionToDelimitedString(defaultMsgs, ", "); // 将数组存放的错误信息以","进行拼接返回
        return JsonVos.error(msg);
    }

    // 处理后端校验ConstraintViolationException抛出的异常(非model参数校验)
    private JsonVo handle(ConstraintViolationException cve) {
        List<String> msgs = Streams.map(cve.getConstraintViolations(), ConstraintViolation::getMessage);
        String msg = StringUtils.collectionToDelimitedString(msgs, ", ");
        return JsonVos.error(msg);
    }
}
```

**注意：** 因为ErrorFilter拦截到的是ServletException（是把CommonException包装后的异常），所以在上述代码中，会单独处理这个异常（不是前面的四种）。

**那如何将ErrorFilter设置为Filter1呢？**
```
@Configuration
public class WebCfg implements WebMvcConfigurer {
    @Autowired
    private JkProperties properties;

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        // /**表示对所有的路径开放全局跨域访问权限
        registry.addMapping("/**")
                // 开放哪些IP、端口、域名的访问权限
                .allowedOrigins(properties.getCfg().getCorsOrigins())
                // 是否允许发送Cookie信息
                .allowCredentials(true)
                // 哪些HTTP方法允许跨域访问
                .allowedMethods("GET", "POST");
    }

    // 在这里注册SpringBoot环境并设置ErrorFilter为Filter1
    @Bean
    public FilterRegistrationBean<Filter> filterRegistrationBean() {
        FilterRegistrationBean<Filter> bean = new FilterRegistrationBean<>();
        // 设置Filter
        bean.setFilter(new ErrorFilter());
        bean.addUrlPatterns("/*");
        // 最高权限: HIGHEST_PRECEDENCE（最小值）
        bean.setOrder(Ordered.HIGHEST_PRECEDENCE);
        return bean;
    }
}
```

### 安全隐患

> token如果是被设计成在规定时间内不会改变，一旦在请求过程中被人为拦截，会造成恶劣影响！

**解决：** 若客户端发送请求（带token）后，服务器发现过期了，会返回一个提示：“token过期了”，然后客户端自动重新发送一个请求来刷新token（客户端获取一个新的token），服务器返回新的token，下次再请求时服务器返回数据！

**如何获取一个新的token？  双token**   
当客户端请求后，服务器会返回两个token：`accessToken`（如7天用于访问，平时请求就带这个token），`refreshToken`（如30天，用于`accessToken`过期后进行刷新，即当7天的`accessToken`过期后，只需发送请求到服务器想要新的7天`accessToken`，这时就需要带上这个`refreshToken`，由此得到一个新的`accessToken`），需要注意的是：向服务器请求数据时，只认`accessToken`才能完成数据返回！

**为什么要设置多个token？**
因为accessToken会频繁请求数据时被发送，所以被人为恶意拦截的概率会很大，若请求的数据并未加密时，会产生较大影响，所以说accessToken的有效期相对较短。

**总结：** 当然了也可以直接设置token多少天过期，期间的token信息不会改变，但是token信息是在我们自己服务器缓存中，当检测到一个token访问数据库过于频繁，IP地址变化等异常情况访问，这是便可以强制`caches.removeToken()`，使其下线！

**Token是如何进入并操作权限的？**

首先会在自定义Filter：TokenFilter进行验证用户的合法性、是否有相关权限（**鉴权**）
```
/**
 * 添加一些自定义Filter
 * 作用：验证用户的合法性、是否有相关权限
 */
@Slf4j
public class TokenFilter extends AccessControlFilter {
    public static final String HEADER_TOKEN = "Token"; // 客户端发送的请求头

    /**
     * 当请求被TokenFilter拦截时，就会调用这个方法
     * 可以在这个方法中做初步判断(放行后的其他请求回来到这，但是也有通过请求类型（POST/GET/OPTIONS等）来看是否放行)
     *
     * 如果返回true：允许访问。可以进入下一个链条调用（比如Filter、拦截器、控制器等）
     * 如果返回false：不允许访问。会进入onAccessDenied方法，不会进入下一个链条调用（比如Filter、拦截器、控制器等）
     */
    @Override
    protected boolean isAccessAllowed(ServletRequest servletRequest, ServletResponse servletResponse, Object o) throws Exception {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
//        // 放行所有的OPTIONS请求
//        return "OPTIONS".equals(request.getMethod());

        log.debug("TokenFilter - isAccessAllowed - " + request.getRequestURI());
        return false;
    }

    /**
     * 当isAccessAllowed返回false时，就会调用这个方法
     * ☆在这个方法中进行token的校验
     *
     * 如果返回true：允许访问。可以进入下一个链条调用（比如Filter、拦截器、控制器等）
     * 如果返回false：不允许访问。不会进入下一个链条调用（比如Filter、拦截器、控制器等）
     */
    @Override
    protected boolean onAccessDenied(ServletRequest servletRequest, ServletResponse servletResponse) throws Exception {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        log.debug("TokenFilter - onAccessDenied - " + request.getRequestURI());

        // 在请求头中取出Token
        String token = request.getHeader(HEADER_TOKEN);

        // 如果没有Token
        if (token == null) {
            return JsonVos.raise(CodeMsg.NO_TOKEN);
        }

        // 有token但如果Token过期了(在缓存Caches中看是否有token)
        if (Caches.getToken(token) == null) {
            return JsonVos.raise(CodeMsg.TOKEN_EXPIRED);
        }

        log.debug("TokenFilter - onAccessDenied - " + token);

        // 鉴权（进入Realm）
        // 这里调用login，并不是“登录”的意思，是为了触发Realm的相应方法去加载用户的角色、权限信息，以便鉴定权限
        SecurityUtils.getSubject().login(new Token(token));
        return true;
    }
}
```

再来到doGetAuthorizationInfo认证，再将token取出来包装成SimpleAuthorizationInfo（通过认证）
```
// 自定义Realm
@Slf4j
public class TokenRealm extends AuthorizingRealm {

    public TokenRealm(TokenMatcher matcher) {
        super(matcher);
    }

    // 这里是判断是不是我们自定义的token，如果不是就不会执行认证和权限操作
    @Override
    public boolean supports(AuthenticationToken token) {
        log.debug("TokenRealm - supports - {}", token);
        return token instanceof Token;
    }

    // 再操作权限信息
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        // 拿到当前登录用户的token
        String token = (String) principals.getPrimaryPrincipal(); // principals是已经认证通过了的principals
        log.debug("TokenRealm - doGetAuthorizationInfo - {}", token);

        // 根据token查找用户的角色、权限
        SysUserDto user = Caches.getToken(token);

        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        List<SysRole> roles = user.getRoles();
        if (CollectionUtils.isEmpty(roles)) return info;

        // 添加角色
        for (SysRole role : roles) {
            info.addRole(role.getName());
        }

        List<SysResource> resources = user.getResources();
        if (CollectionUtils.isEmpty(resources)) return info;
        // 添加权限
        for (SysResource resource : resources) {
            info.addStringPermission(resource.getPermission());
        }
        return info;
}
```

### 缓存中的权限信息
> **权限信息不必每次请求都去查询数据库，可以将权限信息放在缓存中。 而这样存在问题：若超级用户修改了一个用户的权限，而上述说把权限放在缓存中，所以修改后是无法第一时间作出改变的。**

**解决：** 一旦修改一个用户的权限，直接将该用户信息在缓存中删除，需要重新登录。
```
@Service
@Transactional
public class SysUserServiceImpl extends ServiceImpl<SysUserMapper, SysUser> implements SysUserService {
    @Autowired
    private SysUserRoleService userRoleService;
    @Autowired
    private SysRoleService roleService;
    @Autowired
    private SysResourceService resourceService;

    // 保存用户中的角色信息的特殊saveOrUpdate方法
    @Override
    public boolean saveOrUpdate(SysUserReqVo reqVo) {
        // 转成PO
        SysUser po = MapStructs.INSTANCE.reqVo2po(reqVo);

        // 保存用户信息
        if (!saveOrUpdate(po)) return false;

        Integer id = reqVo.getId();
        if (id != null && id > 0) { // 如果是做更新(添加时，不用删除角色信息，本身是没有信息的)
            // 将更新成功的用户从缓存中移除（让token失效，用户必须重新登录）：因为用户信息是放到缓存中的
            Caches.removeToken(Caches.get(id));

            // 删除当前用户的所有角色信息(重点:在编辑操作时，无论用户怎么操作，就先把之前的角色信息全部删掉，提交时再保存当前操作完后的角色信息)
            userRoleService.removeByUserId(reqVo.getId());
        }

        // 保存角色信息
        String roleIdsStr = reqVo.getRoleIds();
        if (Strings.isEmpty(roleIdsStr)) return true; // 若角色id不填，为空串时

        String[] roleIds = roleIdsStr.split(",");
        List<SysUserRole> userRoles = new ArrayList<>();
        Integer userId = po.getId();
        for (String roleId : roleIds) { // 构建SysUserRole对象，对应数据库的sys_user_role表的两条记录
            SysUserRole userRole = new SysUserRole();
            userRole.setUserId(userId);
            userRole.setRoleId(Short.parseShort(roleId));
            // 将userRole中两个属性(userId,roleId)放到集合userRoles中，并作批量保存saveBatch处理
            userRoles.add(userRole);
        }
        return userRoleService.saveBatch(userRoles);
    }
}
```

### 物理删除和逻辑删除

物理删除：真正从数据库中删除了，永久消失了；
逻辑删除（假删除）：数据库还留在数据库中，只是对于用户来说，数据被删除了；

**如何实现逻辑删除？**
增加一个字段用于标识数据是否被删除！

![QQ图片20211004210727.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211004210727-d06f72cdcaeb49d9abd6cd9ee77cd276.png)

由上图：
1. 删除之后deleted变为1（默认为0）；
2. 以后查询操作时，默认在后面加上条件deleted=0（为0表示没有被删除的信息）；
3. 更新也是更新deleted=0的信息；

而以上操作，MyBatis-Plus可以简单标记：

![QQ图片20211004211045.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211004211045-22e93c3f48744f9ea6da59a82aa5b718.png)

- 在局部配置中若设置了select=false时，是无法直接select查询到的；
- 自定义SQL语句时，yBatis-Plus并不会自动加上逻辑删除相关的功能，需要自实现逻辑删除：即在编写select语句时后面加上一个deleted=0即可，
- 当然也可以在wapper里加：
```
wrapper.eq("deleted",0);
```
![QQ图片20211004211526.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211004211526-465aa81b4fd0452e9a2711a3e9b8fa83.png)

若所有表都需要逻辑删除，可以直接使用全局配置即可！

### 文件上传：图片、视频
> 方式一：上传文件直接存储在本服务器的硬盘上去：文件数据跟随表单数据一起提交！

> 方式二（推荐）：文件数据先单独提交，从文件服务器获取一个文件uri，文件uri会后面跟随表单数据一起提交！

![QQ图片20211004211929.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211004211929-8a86104762dc41688e92e987a663f23e.png)

总结：
1. 文件请求是在选择后（提交保存前）就发送给了文件服务器，服务器偷偷返回一个文件uri，之后这个uri会同表单一起提交到业务服务器上；
2. 存在一个情况就是：选择了的文件，保存前突然不要了，所以还得再偷偷发送请求到文件服务器表明刚刚的文件不需要了；

### 补充
> **打包成jar包**

可以打包成jar（默认）、war，都需要添加依赖：
```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    <plugins>
<build>
```

![QQ图片20211004212839.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211004212839-9c2b81155f814156893697a8a01d482b.png)
 
> **打包成war包**

![QQ图片20211004212915.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211004212915-b2076477547f4568aed35dbe3dd071b2.png)

![QQ图片20211004212918.png](/upload/2021/10/QQ%E5%9B%BE%E7%89%8720211004212918-917b7124f4894ea3b216680e55961d96.png)

**如何在Tomcat中打开并部署war包？**
- 将打包的war包放到Tomcat文件目录下的webapps中；
- 在Tomcat文件的bin目录下启动Tomcat：startup.bat；
- 访问Swagger的uri；