## **基本使用**
#### **1.企业的流行框架**
SSM：Spring、SpringMVC、MyBatis
Apache Shiro
SpingBoot

#### **2.客户端请求流程**
- 客户端请求会先来到 `控制层Servlet/Controller`，再到 `业务层Service`，最后为 `数据持久层Dao`。
- 数据层框架：纯JDBC -> SpringJDBC + Druid -> MyBatis  所以MyBatis是面向数据层的框架，封装了JDBC的很多操作细节，让开发者大大简化了Dao层的代码；

#### **3.数据库事务**
- **简单概念：** 如果将N个数据库操作(sql语句)放到了同一个事务中，那么这N个操作(sql语句)最终要么全都生效，要么全都不生效。
- **事务的使用：** `START TRANSACTION`;
- **回滚事务：** `ROLLBACK`  只要事务中的一个操作失败，那么其他所有操作都需要回滚（rollback），回到开启事务之前的状态，之前操作无效；
- **提交事务：** `COMMIT` 如果事务中的所有操作都成功了，就提交事务，让这些操作真正生效；
- 回滚事务于提交事务都是结束事务的一种方式；
- JDBC的事务管理：在JDBC中使用 `Connection` 对象来管理事务，其中`setAutoCommit(false/true)` 来开启事务，`prepareStatement(sql)和executeUpdate()`执行sql语句，`commit()` 来提交事务，`rollback()` 来回滚事务；

#### **4.事务的四大特性**
> **原子性（Atomicity）：** 事务作为一个整体被执行，包含在其中的对数据库的操作要么全部被执行，要么都不执行。
> **一致性（Consistency）：** 事务应确保数据库的状态从一个一致状态转变为另一个一致状态。一致状态的含义是数据库中的数据应满足完整性约束。
> **隔离性（Isolation）：** 多个事务并发执行时，一个事务的执行不应影响其他事务的执行。
> **持久性（Durability）：** 已被提交的事务对数据库的修改应该永久保存在数据库中

#### **5.如何使用MyBatis？**
- 1. **依赖：** mybatis、mysql-connector-java(MySQL数据库驱动包)
- 2. **创建核心配置文件mybatis-config.xml:** 核心配置的主要标签就是`<environments/>`、`<plugins/>`、`<typeAliases/>`、`<settings/>`、`<properties/>`
- 3. **在 `<environments/>` 中**，我们可以添加属性environment来标记多场景下使用哪个环境id（在此标签中可以配置多个环境），**transactionManager** 属性作用是标明采用JDBC事务管理方法来管理事务，而主要 **dataSource** 可以标明哪种方式管理连接，一般采用连接池（而连接池可以是POOLED类型或者DRUID类型），而以下就是配置连接 `<property name="xxx" value="xxx"/>`这里的name一般为 `driver`、`url`、`username`、`password`等基础配置，还有druid连接池的initialSize、maxActive、maxWait等。
- 4. 而在 **`<plugins/>`** 标签中可以配置MyBatis分页插件，可以就简化了查询分页的复杂操作。
- 5. **`<typeAliases/>`** 标签可以配置别名，如 `<typeAlias type="com.ws.bean.Skill" alias="skill" />`
- 6. **<settings/>** 标签一般为其他设置，如数据库名到Java中Bean对象的转换：**数据库：my_first_name -> Java：myFirstName** （开启驼峰命名自动映射）
- 7. 若想通过JDBC来管理事务并访问该表，需要添加一个映射xml文件，该文件就是为了实现Skill JavaBean与数据库skill表之间的映射。
- 8. xxx.xml映射文件的根标签为 `<mapper/>` 在这里就可以编写sql语句，而且可以编写动态sql语句，新建的映射文件可以放在resources下的mapper文件里。如 resources/mapper/skill.xml
![QQ图片20210918102846.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210918102846-b06180a499ad488abb912fb8e6ea0279.png)

- 9. 当然了，也要mybatis配置文件中的 `<mapper/>` 添加实体映射文件路径。
- 10. 如何使用java代码加载使用？ 用工厂构建器构建工厂后创建Session并执行对应的sql语句

**总结：** 拿到数据库的数据的流程？
> 先在pom.xml中配置mybatis、mysql、junit等的依赖;

> 在mybatis-config.xml文件中配置环境：JDBC事务、POOLED连接池等;

> 创建Bean对象并配置Bean与对应数据库表之间的映射文件;

> 再在mybatis配置文件中添加映射文件路径供统一加载;

> 写测试代码，用工厂构建器构建工厂后创建Session并执行对应的sql语句;

#### **6.传参**
- 在mapper映射文件中添加select标签，其中用 `#{xxx}` 是参数在mapper.xml文件中sql语句的占位符。
- 注意：${}是直接文本替换，而#{}为预编译传值，可以防止SQL注入，所以我们一般使用#{};
![QQ图片20210918105358.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210918105358-5f33313977214c9fade409bdd52f575a.png)

- 同理传入多个参数也是一样
![QQ图片20210918105512.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210918105512-65527d0160d14f46bf2ffef1485720cc.png)

- 以上可知：只需创建一次工厂即可，后续直接MyBatises.openSession()获取连接session，其中参数设置为true时表示自动提交事务，否则需要手动提交，而查询操作不需要提交事务，只有对数据库进行操作的才要提交事务。

- 所以在当用'$'符号时，不能是由用户传进来的，但也不能全用#符号进行传参，'$'存在即合理，在直接替换的地方可以使用$，而#是要进行预编译的。


> 在输出打印sql语句时，可以直接在mybatis配置文件中配置，也可以使用第三方库，如logback等，这样便可以看见更为详细的日志输出。


#### **7.多表查询（关联表）**
- 将resultType设置为Map、HashMap、LinkedHashMap等，其中LinkedHashMap可以实现添加顺序打印
![QQ图片20210918144844.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210918144844-85e57c3bcb304f0c99d3972a2fa3bf22.png)

> 映射文件中的输入类型parameterType与输出resultType在后续的定义各种Object对象会有所区分，在这里都会一样的po对象

![QQ图片20210918145129.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210918145129-df1a528d54904291bd98bb19be430432.png)
- 这样便可以实现多表查询


## **MyBatis的增删改**

#### **1.添加（增）**
- 在xml中，添加的标签是 `<insert/>` 添加的输入参数类型是Bean对象：

```mysql
    <!--添加（增）-->
    <insert id="insert" parameterType="com.ws.bean.Skill">
        INSERT INTO skill(name, level) VALUES(#{name}, #{level})
    </insert>
```

> 注意 `resultType（输出）` 与 `parameterType（输入）` 的区别

- 在测试代码中，需要先获取连接session，再新建Bean对象传入参数，最后调用insert()函数执行添加sql语句

```java
    // 增
    @Test
    public void insert() throws Exception {
        try (SqlSession session = MyBatises.openSession(true)) {
            Skill skill = new Skill("iOS", 8);
            session.insert("skill.insert", skill);
        }
    }
```

> 这里的 `MyBatises.openSession(true)` 中设置为true，表示执行完后自动提交事务，为下面的inser函数参数处理Bean对象外，还有就是标明执行的是xml文件的id为insert的sql语句。

#### **若想在插入数据后，如何直接返回对应的插入id值？** --需要主键设置
- 这是是不能再写一条sql语句去查询刚刚插入到数据库的数据id值，因为这样存在多线程问题！
- 第一种方法就是设置新插入记录的主键id到参数对象中:

xml的sql代码
```mysql
    <insert id="insert2" parameterType="com.ws.bean.Skill">
        INSERT INTO skill(name, level) VALUES(#{name}, #{level})
        <selectKey resultType="int" keyProperty="id" order="AFTER">
            SELECT LAST_INSERT_ID()
        </selectKey>
    </insert>
    <!--使用selectKey标签：resulType返回值id为int类型，keyProperty为属性名id，order=AFTER 设置返回id的操作是在上面sql语句执行之后，SELECT LAST_INSERT_ID() 是查询最后一次插入的id值-->
```

 java代码：
```java
    // 设置新插入记录的主键id到参数对象中
    @Test
    public void insert2() throws Exception {
        try (SqlSession session = MyBatises.openSession()) {
            Skill skill = new Skill("Android", 9);
            session.insert("skill.insert2", skill);
            System.out.println(skill.getId()); // 插入后拿到对应id
            session.commit(); // 手动提交事务管理
        }
    }
```

- 第二种方法就是使用useGeneratedKeys，keyProperty属性，但方法需要数据库驱动支持，如适用于MySQL,不适用Oracle，java代码相似。

```mysql
    <insert id="insert3"
            useGeneratedKeys="true"
            keyProperty="id"
            parameterType="com.ws.bean.Skill">
        INSERT INTO skill(name, level) VALUES(#{name}, #{level})
    </insert>
```

#### **2.更新与删除**

xml
```mysql
    <!-- 更新：在传入id参数后，更新值 -->
    <update id="update" parameterType="com.ws.bean.Skill">
        UPDATE skill SET name = #{name}, level = #{level} WHERE id = #{id}
    </update>

    <!-- 删除：只需要id -->
    <delete id="delete" parameterType="int">
        DELETE FROM skill WHERE id = #{id}
    </delete>
```

java

```java
    // 更新
    @Test
    public void update() throws Exception {
        try (SqlSession session = MyBatises.openSession()) {
            // 将id值为2的数据改为Java(name)和666(level)
            Skill skill = new Skill("Java", 666);
            skill.setId(2);

            session.update("skill.update", skill);
            session.commit();
        }
    }

    // 删除
    @Test
    public void delete() throws Exception {
        try (SqlSession session = MyBatises.openSession()) {

            session.delete("skill.delete", 1);

            session.commit();
        }
    }
```

#### **3.动态sql**
 
> **if标签**
xml
```mysql
    <!--动态生成SQL语句-->
    <select id="dynamicSQL1" parameterType="Map" resultType="com.ws.bean.Skill">
        SELECT * FROM skill WHERE 1 = 1

        <if test="id != null">
            AND id > #{id}
        </if>

        <if test="name != null">
            AND name LIKE #{name}
        </if>

        <if test="level != null">
            AND level &lt; #{level}
        </if>
    </select>
```
- 由上可以看出：动态生成sql十分方便，而在javaEE中使用springJDBC是十分的麻烦！ `SELECT * FROM skill WHERE 1 = 1` 后便是根据传入的参数，动态生成sql语句的逻辑；`if test` 测试条件成功之后拼接条件sql语句！

java
```java
    // 动态sql
    @Test
    public void dynamicSQL() throws Exception {
        try (SqlSession session = MyBatises.openSession()) {
            Map<String, Object> param = new HashMap<>();
            param.put("id", 2);
            param.put("name", "%s%");
            param.put("level", 600);

            /*
            sql的存在形式:
            SELECT * FROM skill WHERE 1 = 1
            AND id > #{id}
            AND name LIKE #{name}
            AND level < #{level}
  
            意思就是：查询id大于2且name中包含 s 且 level小于600的满足条件的数据
             */

            List<Skill> skills = session.selectList("skill.dynamicSQL1", param);

            for (Skill skill : skills) {
                System.out.println(skill);
            }
        }
    }

```

> **where标签**

xml
```mysql
    <select id="dynamicSQL2" parameterType="Map" resultType="com.ws.bean.Skill">
        SELECT * FROM skill

        <where>
            <if test="id != null">
                AND id > #{id}
            </if>

            <if test="name != null">
                AND name LIKE #{name}
            </if>

            <if test="level != null">
                AND level &lt; #{level}
            </if>
        </where>
    </select>
```
- java代码一样，只是where和if标签的写法有一点不同：由结果对比可知，生成的sql语句没有 1=1，更接近于真正的sql语句
- 由上面共同的java代码可知：在 `MyBatises.openSession()` 中并没有设置为true，也没有在后面手动提交事务，为什么？ 因为查询并不用提交事务，只有那些修改数据库数据的操作才会提交事务。

#### **4.批量处理**
> **批量添加**

xml
```mysql
    <insert id="batchInsert"
            useGeneratedKeys="true"
            keyProperty="id"
            parameterType="List">
        INSERT INTO skill(name, level) VALUES
        <!-- 遍历list数组，拿出里面全部的skill对象;separator表示分隔符, -->
        <foreach collection="list" item="skill" separator=",">
            (#{skill.name}, #{skill.level})
        </foreach>
    </insert>
```
java
```java
    // 批量添加
    @Test
    public void batchInsert() throws Exception {
        try (SqlSession session = MyBatises.openSession()) {

            List<Skill> skills = new ArrayList<>();
            skills.add(new Skill("Java5", 11));
            skills.add(new Skill("Java6", 22));
            skills.add(new Skill("Java7", 33));
            skills.add(new Skill("Java8", 44));
            skills.add(new Skill("Java9", 54));
            skills.add(new Skill("Java10", 64));

            // 这里的insert是有一个int类型的返回值的，表示影响了多少条语句(上面表示影响了4条)
            session.insert("skill.batchInsert", skills);

            for (Skill skill : skills) {
                System.out.println(skill.getId());
            }

            session.commit();
        }
    }
```

> **批量删除**

xml
```mysql
    <!--传进来的参数是List-->
    <delete id="batchDelete" parameterType="List">
        DELETE FROM skill WHERE id IN (
        <foreach collection="list" item="id" separator=",">
            #{id}
        </foreach>
        )
    </delete>

    <!--传进来的参数是数组-->
    <delete id="batchDelete2">
        DELETE FROM skill WHERE id IN
        <foreach collection="array"
                 item="id"
                 open="("
                 close=")"
                 separator=",">
            #{id}
        </foreach>
    </delete>
```

java
```java
    // 批量删除
    @Test
    public void batchDelete() throws Exception {
        try (SqlSession session = MyBatises.openSession()) {

	    // List<Integer> ids = new ArrayList<>();
	    // ids.add(3);
	    // ids.add(7);
	    // ids.add(10);
	    // ids.add(11);

            Integer[] ids = {3, 7, 10, 11};

            session.insert("skill.batchDelete2", ids);

            session.commit();
        }
    }
```

**注意：** 
- 批量添加的效率比【多次单个添加】要高，但是它无法使用 `<selectedKey>` 获取新插入记录的主键，但可以使用 `useGeneratedKeys` 获取主键；
- 批量操作生成的SQL语句可能会比较长，有可能会超过数据库的限制；
- 如果传进来的参数是List，collection属性值为list就可以遍历这个Lisy;
- 如果传进来的参数是数组，collection属性值为array就可以遍历这个数组；
- **如何解决批量操作生成的sql语句可能超过数据库限制的问题？**
设置数据库的操作内存大小；分批次操作，如执行10000条生气了语句，可以分10次1000的操作。


## **MyBatis的连接池-分页**

#### **1.设置Druid连接池**
- 若使用mybatis内置的连接池则用 `POLLED`，而想用第三方库 `Druid/C3PO` 等需要改变；
- 添加依赖druid，新建一个类并必须继承 `UnpooledDataSourceFactory` 这个mybatis内置类；
- 同时在构造方法中设置数据源 `datasource` :
```java
public class DruidDataSourceFactory extends UnpooledDataSourceFactory {
    public DruidDataSourceFactory() {
        this.dataSource = new DruidDataSource();
    }
}
```
- 读取：第一在创建druid.properies文件配置属性，并在mybatis配置文件直接读取（`<properties resource="druid.properties" />`）; 第二直接在mybatis配置文件中将POLLED换成DRUID，依次设置数据库以及连接的值。

```java
	#druid.properties

	dev.driverClass=com.mysql.jdbc.Driver
	dev.url=jdbc:mysql://localhost:3306/test_mybatis
	dev.username=root
	dev.password=root
	dev.initialSize=5
	dev.maxActive=10
	dev.maxWait=5000
```
```java
    // mybatis-config.xml

    <properties resource="druid.properties" />
    <environments default="development">
        <!-- 开发环境（调试阶段3） -->
        <environment id="development">
            <!-- 采用JDBC的事务管理方法 -->
            <transactionManager type="JDBC" />

            <!-- POOLED代表采取连接池的方式管理连接 -->
            <dataSource type="DRUID">
                <property name="driverClass" value="${dev.driverClass}"/>
                <property name="url" value="${dev.url}"/>
                <property name="username" value="${dev.username}"/>
                <property name="password" value="${dev.password}"/>
                <property name="initialSize" value="${dev.initialSize}"/>
                <property name="maxActive" value="${dev.maxActive}"/>
                <property name="maxWait" value="${dev.maxWait}"/>
            </dataSource>
        </environment>
    </environments>
```
- 通过以上操作，把类名设置到dataSource的Type属性上，然后拿到这个 `DruidDataSourceFactory` 并创建这个工厂，创建完后就去读取配置文件中的property属性配置信息，最终会将配置信息设置到前面的datasource上去；

#### **2.分页**
![QQ图片20210918173005.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210918173005-c0502a19f19f4098841701b20f4abb4b.png)

> 每个数据库的分页查询语句都不太一样，所以采用PageHelper这个mybatis分页插件，支持多种数据库并简化分页的业务逻辑。

> 解析上图Oracle的查询思路：①先把student里的所有属性，如id、name等查询出来（假设总共数据有30条）；②中间这个第二层select就是为了给30条数据加上行号rn，并筛选出小于20的数据（1~20）；③最外层的select就是对rn大于等于11的筛查，得到11~20的数据。这中方法三次select，效率低！

**为了屏蔽每种数据库执行语句的不同，是代码更具有健壮性，将使用mybatis分页插件 `PageHelper`，它支持多种常用的数据库，可以极大简化分页的业务逻辑。如何使用？**

- 添加依赖
- 在mybatis-config.xml中配置插件
```xml
    <!-- MyBatis分页插件 -->
    <plugins>
        <plugin interceptor="com.github.pagehelper.PageInterceptor">
            <!-- true代表分页合理化：pageNum <= 0就会自动获取第1页，pageNum > pages就会自动获取最后1页 -->
            <property name="reasonable" value="true"/>
        </plugin>
    </plugins>
```
- 使用方法：直接查询第2页的5条数据，即 6~10
```java
    @Test
    public void page() throws Exception {
        try (SqlSession session = MyBatises.openSession()) {
            PageHelper.startPage(2, 5);
            List<Skill> skills = session.selectList("skill.list");
            for (Skill skill : skills) {
                System.out.println(skill);
            }
        }
    }
```

## **MyBatis的多表查询与延迟加载**

### **1.多表查询**

- 数据库的表之间可能存在一定的关系：一对多\多对一、一对一、多对多

> **多表关系：一对一（一个人对应一张身份证，反之一样）**

![QQ图片20210919091702.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210919091702-fed5228c74774da3a28c498e18ee2fb6.png)


> **多表关系：一对多\多对一（一个人对应多张银行卡）**

![QQ图片20210919091552.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210919091552-c6760435bae34863b1f1c0c4f23144c7.png)

> **多表关系：多对多（多个人对应多个工作岗位）**

![QQ图片20210919092329.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210919092329-6a6b2201339243bcb41a2a2909ccc2c9.png)

- 多表之间的xml映射关系：

person.xml（只是是用person去关联身份证、银行卡以及工作）
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="person">
    <sql id="sqlListAll">
        SELECT
            p.*,
            ic.id id_card_id,
            ic.no id_card_no,
            ic.address id_card_address,
            bc.id bank_card_id,
            bc.no bank_card_no,
            bc.amout bank_card_amout,
            j.id job_id,
            j.name job_name,
            j.duty job_duty
        FROM
            person p
        <!--这里的idcard不用左连接，直接内连接，因为每一个人只有一个身份证-->
        JOIN id_card ic ON p.id = ic.person_id
        LEFT JOIN bank_card bc ON p.id = bc.person_id
        LEFT JOIN person_job pj ON p.id = pj.person_id
        LEFT JOIN job j ON j.id = pj.job_id
    </sql>

    <!--字段映射-->
    <resultMap id="rmList" type="com.ws.bean.Person">
        <id property="id" column="id"/>
        <result property="name" column="name"/>
        <!--idCard的映射-->
        <association property="idCard"
                     javaType="com.ws.bean.IdCard">
            <id property="id" column="id_card_id"/>
            <result property="no" column="id_card_no"/>
            <result property="address" column="id_card_address"/>
        </association>
        <!--bankCards的映射-->
        <collection property="bankCards" ofType="com.ws.bean.BankCard">
            <id property="id" column="bank_card_id"/>
            <result property="no" column="bank_card_no"/>
            <result property="amout" column="bank_card_amout"/>
        </collection>
        <!--jobs的映射-->
        <collection property="jobs" ofType="com.ws.bean.Job">
            <id property="id" column="job_id"/>
            <result property="name" column="job_name"/>
            <result property="duty" column="job_duty"/>
        </collection>
    </resultMap>

    <select id="list" resultMap="rmList">
        <include refid="sqlListAll"/>
    </select>

    <select id="get" parameterType="int" resultMap="rmList">
        <include refid="sqlListAll"/> WHERE p.id = #{id}
    </select>
</mapper>
```

**注意：** 
- 在一对一关系中，字段映射时使用标签 `<association/>`，而一对多、多对多是使用 `<collection/>`，因为一对多、多对多产生的数据是集合了。
- 特别注意字段映射时自己的属性与关联表的属性的映射格式写法。
- 对数据库操作的MyBatis和上节是一样的：这里需要在下面配置所有的实体映射文件的路径！
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="druid.properties" />

    <!-- 其他设置 -->
    <settings>
        <!-- 数据库：my_first_name -> Java：myFirstName -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>

    <!-- 别名 -->
    <typeAliases>
        <!-- 一旦设置了别名，它是不区分大小写的 -->
        <typeAlias type="com.ws.common.DruidDataSourceFactory" alias="druid" />
    </typeAliases>

    <!-- 环境 -->
    <environments default="development">
        <!-- 开发环境（调试阶段） -->
        <environment id="development">
            <!-- 采用JDBC的事务管理方法 -->
            <transactionManager type="JDBC" />

            <!-- POOLED代表采取连接池的方式管理连接 -->
            <dataSource type="DRUID">
                <property name="driverClass" value="${dev.driverClass}"/>
                <property name="url" value="${dev.url}"/>
                <property name="username" value="${dev.username}"/>
                <property name="password" value="${dev.password}"/>
                <property name="initialSize" value="${dev.initialSize}"/>
                <property name="maxActive" value="${dev.maxActive}"/>
                <property name="maxWait" value="${dev.maxWait}"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 添加实体映射文件的路径 -->
    <mappers>
        <mapper resource="mappers/person.xml" />
        <mapper resource="mappers/idCard.xml" />
        <mapper resource="mappers/bankCard.xml" />
        <mapper resource="mappers/job.xml" />
    </mappers>
</configuration>
```

#### **反过来：可以用person去关联其他表，也可以分别用其他表去关联person**

idCard
```xml
<mapper namespace="idCard">
    <sql id="sqlListAll">
        SELECT
            ic.*,
            p.name person_name
        FROM
            id_card ic
        JOIN person p ON p.id = ic.person_id
    </sql>

    <!--    映射-->
    <resultMap id="rmList" type="com.ws.bean.IdCard">
        <id property="id" column="id"/>
        <result property="no" column="no"/>
        <result property="address" column="address"/>
        <association property="person" javaType="com.ws.bean.Person"> <!--关联标签-->
            <id property="id" column="person_id"/>
            <result property="name" column="person_name"/>
        </association>
    </resultMap>

    <select id="list" resultMap="rmList">
        <include refid="sqlListAll"></include>
    </select>

    <select id="get" parameterType="int" resultMap="rmList">
        <include refid="sqlListAll"/> WHERE ic.id = #{id}
    </select>
```

bankCard
```xml
<mapper namespace="bankCard">
    <sql id="sqlListAll">
        SELECT
            bc.*,
            p.name person_name
        FROM
            bank_card bc
        JOIN person p ON p.id = bc.person_id
    </sql>

    <resultMap id="rmList" type="com.ws.bean.BankCard">
        <id property="id" column="id"/>
        <result property="no" column="no"/>
        <result property="amout" column="amout"/>
        <association property="person" javaType="com.ws.bean.Person">
            <id property="id" column="person_id"/>
            <result property="name" column="person_name"/>
        </association>
    </resultMap>

    <select id="list" resultMap="rmList">
        <include refid="sqlListAll"/>
    </select>

    <select id="get" parameterType="int" resultMap="rmList">
        <include refid="sqlListAll"/> WHERE bc.id = #{id}
    </select>
</mapper>
```
job
```xml
<mapper namespace="job">
    <sql id="sqlListAll">
        SELECT
            j.*,
            p.id person_id,
            p.name person_name
        FROM
            job j
        LEFT JOIN person_job pj ON j.id = pj.job_id
        LEFT JOIN person p ON p.id = pj.person_id
    </sql>

    <resultMap id="rmList" type="com.ws.bean.Job">
        <id property="id" column="id"/>
        <result property="name" column="name"/>
        <result property="duty" column="duty"/>
        <collection property="persons" ofType="com.ws.bean.Person">
            <id property="id" column="person_id"/>
            <result property="name" column="person_name"/>
        </collection>
    </resultMap>

    <select id="list" resultMap="rmList">
        <include refid="sqlListAll"/>
    </select>

    <select id="get" parameterType="int" resultMap="rmList">
        <include refid="sqlListAll"/> WHERE j.id = #{id}
    </select>
</mapper>
```

测试java代码：多对一与多对多也是同样的
```java
public class OneToOne {
    @Test
    public void personList() {
        try (SqlSession session = MyBatises.openSession()) {
            List<Person> persons = session.selectList("person.list");
            System.out.println(persons);
        }
    }

    @Test
    public void personGet() {
        try (SqlSession session = MyBatises.openSession()) {
            Person person = session.selectOne("person.get", 1);
            System.out.println(person);
        }
    }

    @Test
    public void idCardList() {
        try (SqlSession session = MyBatises.openSession()) {
            List<IdCard> idCards = session.selectList("idCard.list");
            System.out.println(idCards);
        }
    }

    @Test
    public void idCardGet() {
        try (SqlSession session = MyBatises.openSession()) {
            IdCard idCard = session.selectOne("idCard.get", 1);
            System.out.println(idCard);
        }
    }
}

```

### **2.延迟加载**

> **由上面的一对多、多对多中的banlCard与job可知：会存在一个人对应多张银行卡和职业，如果每次直接全部查出十分耗费资源，希望只是在需要的时候查询出想要的部分结果比较好！**

- 关联对象（association、collection）可以实现延迟加载。
-  实现延迟加载的步骤，还需要设置以下属性：`fetchType:` 设置为lazy；`select：`指定一个select标签的id，用于查询需要延迟加载的记录；`column：`指定一个列的值，作为调用select查询时传入的参数值；在很多时候，都会给 `collection` 设置延迟加载
- 延迟加载一般分为全局延迟加载和局部延迟加载：

**全局延迟加载开关**
- 全局延迟加载开关，在mybatis-config.xml的setting中配置;
```xml
    <!-- 其他设置 -->
    <settings>
	<!-- true代表所有设置了select属性的关联对象都延迟加载 -->
        <setting name="lazyLoadingEnabled" value="true"/>
    </settings>

    <!--true:所有设置了select属性的关联对象默认懒加载，但可以通过fetchType覆盖这个全局设置-->
```

**局部延迟加载开关**
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="person">
    <sql id="sqlListAll">
        SELECT * FROM person
    </sql>

    <resultMap id="rmList" type="com.ws.bean.Person">
        <id property="id" column="id"/>
        <result property="name" column="name"/>
        <association property="idCard"
                     fetchType="lazy"
                     column="id"
                     select="idCard.getByPerson"
                     javaType="com.ws.bean.IdCard"/>
        <collection property="bankCards"
                    fetchType="lazy"
                    column="id"
                    select="bankCard.listByPerson"
                    ofType="com.ws.bean.BankCard"/>
        <collection property="jobs"
                    fetchType="lazy"
                    column="id"
                    select="job.listByPerson"
                    ofType="com.ws.bean.Job"/>
    </resultMap>

    <select id="list" resultMap="rmList">
        <include refid="sqlListAll"/>
    </select>

    <select id="get" parameterType="int" resultMap="rmList">
        <include refid="sqlListAll"/> WHERE id = #{id}
    </select>
</mapper>
```

**注意：** 以上代码就是在各自xml中实现全部延迟加载，但就算在mybatis的配置文件中设置了全部延迟加载，在各自的映射文件中也可以用 `fetchType="eager"` 进行覆盖。 **在很多时候，都会给 `collection` 设置延迟加载**。


## **MyBatis的缓存与构造方法**

### **1.缓存**

> **缓存是指为了减少数据库直接访问次数、提高访问效率，而临时存储在内存中的数据；**

> **适合存放到缓存中的数据是经常查询、不经常改变、数据的正确性对最终结果影响不大；**

> **举例一些不适合放到缓存中的数据：商品库存、股票 黄金的价格、汇率；**

> **MyBatis的缓存分为一级缓存、二级缓存，用于缓存select的结果；**

> **缓存中的信息并没有数据库的数据新；**

#### **为什么要使用缓存？**
![QQ图片20210919104847.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210919104847-53faa5409e65496bb002c909bd8c2d5d.png)

#### **为什么从缓存中获取信息更快？**
因为缓存的信息是存储在内存中的，而数据库文件信息是存储在硬盘中，常识可知内存比访问硬盘的速度更快！

#### **测试代码结构**
![QQ图片20210919105150.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210919105150-3c524ea30a94404891f09e37f90a0f86.png)

> **一级缓存是存放到了 `SqlSession` 对象中：由于 mybatis 的自动启动一级缓存，所以同一个Session拿到的数据的内存地址是一样的，即同一个 `SqlSession` 的`select`共享缓存，使用关闭 `SqlSession` 时，缓存也失效了；执行 `insert`、`update`、`delete`、`commit`等方法时，会自动清理一级缓存，由于在很多时候，每次查询用的都是不同的`SqlSession`，所以一级缓存的命中率并不高。**

> **为了提高缓存的命中率，可以考虑开启MyBatis的二级缓存，它是`namespace（mapper）`级别的缓存，同一个namespace下的select共享缓存，默认情况namespace下`insert`、`update`、`delete`执行成功时，会自动清理二级缓存，当调用的 `SqlSession` 方法 `close` 时，会将查询结果放进二级缓存。**

> **可以理解为一级缓存是存储在Session中，而二级缓存是存在sqlSessionFactory全局唯一份。**


```xml
<mapper namespace="job">
    <cache readOnly="false"/>

    <sql id="sqlListAll">
        SELECT * FROM job
    </sql>

    <select id="list" resultType="com.ws.bean.Job">
        <include refid="sqlListAll"/>
    </select>

    <select id="get" parameterType="int" resultType="com.ws.bean.Job">
        <include refid="sqlListAll"/> WHERE id = #{id}
    </select>

    <update id="update" parameterType="com.ws.bean.Job">
        UPDATE job SET name = #{name}, duty = #{duty} WHERE id = #{id}
    </update>
</mapper>
```
由上可知：
- 一级缓存是子标签 `<select/>` 级别的，重新开启一个session查询（`MyBatises.openSession()`）时，同一个select是不能在缓存中拿到的。
- 而二级缓存是namespace级别的（根标签），就算重新打开一个session，再重新查询其里面的同一个select是可以在缓存中拿到的，提高了缓存的命中率。

#### **开启二级缓存**

1. 在mybatis-config.xml中配置
```xml
    <!-- 其他设置 -->
    <settings>
        <!-- 开启二级缓存 -->
        <setting name="cacheEnabled" value="true"/>
    </settings>
```
2. 在映射文件的 `mapper` 中添加 `cache` 标签，默认会缓存映射文件中的所有 `select` 的结果：在上图xml代码中可以看见

```xml
    <cache readOnly="false"/>

    <!--
       cache标签的常用属性有：
	  size:缓存多少个存储结果（单个对象或一个列表）的引用，默认值是1024；
	  eviction:当缓存数量超过size时的清除策略。可选值有LRU（默认值，最近最少使用）、FIFO、SOFT、WEAK；
	  flushInterval:每隔多少毫秒清除一次缓存，默认不会定时清除缓存；
	  readOnly: true代表缓存的是对原对象的引用，false代表缓存的是原对象序列化后的拷贝对象，所以false时要求实现Serializable接口。默认值是false；
   -->
```

> **注意： 在对应映射文件添加 cache 标签，其中属性有 `size（缓存多少个存储结果）`、`eviction（超过size后的清楚策略，默认LRU，重要）`、`flushInterval、readOnly`，其中 `readOnly` 属性值为 true/false，默认为false，设置为true表示是对原对象的引用，后面拿到的都是这一个不变的内存地址，这样是不安全的：若后面对其进行了修改，就会直接修改二级缓存中的内容，而false不是对原对象的引用，而先会对原对象进行 `序列化`，则后面的其他sqlSession拿到的都是对象反序列化生成的，虽然这样拿到的不是同一个内存地址，但还是只访问了一次数据库（false只需在bean中设置可实现序列化，实现接口Serializable）**；

3. 当然了，我们也可以通过属性设置来解决哪些要开启二级缓存！

- 可以通过设置 `useCache` 属性来决定某个select是否需要开启二级缓存；
- 可以通过设置 `flushCache` 属性来决定某个操作之后，是否需要清除缓存；
![QQ图片20210919142101.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210919142101-75f81854d6e7436892ef0e068b64f448.png)
- 上述代码表示：在sava操作之后，`flushCache="false"` 不会清楚缓存；

**注意：** 在我们的上一个知识点延迟加载中说到：就算在全局延迟加载开关在mybatis-config.xml中配置了全部延迟加载，但也是可以在各自映射文件中用 `fetchType="eager"` 来进行覆盖这个全局延迟加载，从而设置为立即加载。 而在这里却不行，如果你关闭了全局二级缓存在 mybatis-config.xml 中设置 `<setting name="cacheEnabled" value="false"/>` ，而就算在各自映射文件的 select中 设置 `useCache=true` 也是不能特殊打开二级缓存的。

### **2.构造方法**

> 指定构造方法1

![QQ图片20210919143704.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210919143704-98035be753764cadbee4265735016001.png)

> 指定构造方法2

![QQ图片20210919143707.png](/upload/2021/09/QQ%E5%9B%BE%E7%89%8720210919143707-7952d44dae044619a663de2a26354b35.png)

## **MyBatis的Dao**

>  **使用MyBatis实现Dao层的几种方式?**

> 1. **自定义dao实现类，在实现中调用SqlSession的相关方法（使用XML）**
> 2. **只定义dao接口类，SqlSession的getMapper方法生成dao的代理对象（使用XML）**
> 3. **只定义dao接口类，SqlSession的getMapper方法生成dao的代理对象（使用注解）**

#### **1.自定义dao实现类，在实现中调用SqlSession的相关方法（使用XML）**

**创建dao接口**
```java
     public interface SkillDao {
    	   boolean save(Skill skill);
    	   boolean update(Skill skill);
    	   boolean remove(Integer id);
    	   Skill get(Integer id);
    	   List<Skill> list();
     }
```

**在xxxDaoImpl中具体实现：这里实际就是调用映射文件中的sql语句**
```java
public class SkillDaoImpl implements SkillDao {
    @Override
    public boolean save(Skill skill) {
        try (SqlSession session = MyBatises.openSession(true)) {
            return session.insert("skill.save", skill) > 0; // 传入skill对象用于保存
        }
    }

    @Override
    public boolean update(Skill skill) {
        try (SqlSession session = MyBatises.openSession(true)) {
            return session.update("skill.update", skill) > 0;
        }
    }

    @Override
    public boolean remove(Integer id) {
        try (SqlSession session = MyBatises.openSession(true)) {
            return session.delete("skill.remove", id) > 0;
        }
    }

    @Override
    public Skill get(Integer id) {
        try (SqlSession session = MyBatises.openSession()) {
            return session.selectOne("skill.get", id);
        }
    }

    @Override
    public List<Skill> list() {
        try (SqlSession session = MyBatises.openSession()) {
            return session.selectList("skill.list");
        }
    }
}
```
**则在映射文件中的sql语句实现**
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="skill">
    <sql id="sqlList">SELECT * FROM skill</sql>

<!--    resultType="Skill"中的Skill是别名："com.ws.bean.Skill"-->
    <select id="get" parameterType="int" resultType="Skill">
        <include refid="sqlList"/> WHERE id = #{id}
    </select>

    <select id="list" resultType="Skill">
        <include refid="sqlList"/>
    </select>

    <update id="update" parameterType="Skill">
        UPDATE skill SET name = #{name}, level = #{level} WHERE id = #{id}
    </update>

    <insert id="save"
            useGeneratedKeys="true" keyProperty="id"
            parameterType="Skill">
        INSERT INTO skill(name, level) VALUES(#{name}, #{level})
    </insert>

    <delete id="remove" parameterType="int">
        DELETE FROM skill WHERE id = #{id}
    </delete>
</mapper>
```

#### **测试代码**
```java
public class SkillTest {
    @Test
    public void get() {
        SkillDao dao = new SkillDaoImpl();
        System.out.println(dao.get(2));
    }

    @Test
    public void list() {
        SkillDao dao = new SkillDaoImpl();
        System.out.println(dao.list());
    }

    @Test
    public void save() {
        SkillDao dao = new SkillDaoImpl();
        Assert.assertTrue(dao.save(new Skill("mj888", 100)));
    }

    @Test
    public void update() {
        SkillDao dao = new SkillDaoImpl();
        Skill skill = new Skill("666", 99);
        skill.setId(16);
        Assert.assertTrue(dao.update(skill));
    }

    @Test
    public void remove() {
        SkillDao dao = new SkillDaoImpl();
        Assert.assertTrue(dao.remove(19));
    }
}
```

> **特别需要注意两者之间的标签对应：`select标签 -> get/list`、`update标签 -> update`、`insert标签 -> save`、`delete标签 -> remove`**

> **这种方式存在代码重复较多，累赘！**

#### **2.只定义dao接口类，SqlSession的getMapper方法生成dao的代理对象（使用XML）**

**创建dao接口：并没有用xxxDaoImpl实现这个接口**
```java
	public interface SkillDao {
    	   boolean save(Skill skill);
    	   boolean update(Skill skill);
    	   boolean remove(Integer id);
    	   Skill get(Integer id);
    	   List<Skill> list();
	}

```

**则在映射文件中的sql语句实现**
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ws.dao.SkillDao">    #这里必须使用全名
    <sql id="sqlList">SELECT * FROM skill</sql>

<!--    指定调用有参构造方法(level,name)-->
    <resultMap id="rmGet" type="Skill">
        <constructor>
            <arg name="level" column="level"/>
            <arg name="name" column="name"/>
        </constructor>
    </resultMap>

    <select id="get" parameterType="int" resultMap="rmGet">
        <include refid="sqlList"/> WHERE id = #{id}
    </select>

    <select id="list" resultType="Skill">
        <include refid="sqlList"/>
    </select>

    <update id="update" parameterType="Skill">
        UPDATE skill SET name = #{name}, level = #{level} WHERE id = #{id}
    </update>

    <insert id="save"
            useGeneratedKeys="true" keyProperty="id"
            parameterType="Skill">
        INSERT INTO skill(name, level) VALUES(#{name}, #{level})
<!--        <selectKey keyProperty="id" resultType="int" order="AFTER">-->
<!--            SELECT LAST_INSERT_ID()-->
<!--        </selectKey>-->
    </insert>

    <delete id="remove" parameterType="int">
        DELETE FROM skill WHERE id = #{id}
    </delete>
</mapper>
```

#### **测试代码**
```java
public class SkillTest {

    @Test
    public void get() {
        try (SqlSession session = MyBatises.openSession()) {
            // 生成SkillDao的代理对象(外面并没有具体实现SkillDaoImpl)
            SkillDao dao = session.getMapper(SkillDao.class);
            System.out.println(dao.get(2));
        }
    }

    @Test
    public void list() {
        try (SqlSession session = MyBatises.openSession()) {
            // 生成SkillDao的代理对象
            SkillDao dao = session.getMapper(SkillDao.class);
            System.out.println(dao.list());
        }
    }

    @Test
    public void save() {
        try (SqlSession session = MyBatises.openSession(true)) {
            // 生成SkillDao的代理对象
            SkillDao dao = session.getMapper(SkillDao.class);
            Assert.assertTrue(dao.save(new Skill("mj888", 100)));
        }
    }

    @Test
    public void update() {
        try (SqlSession session = MyBatises.openSession(true)) {
            // 生成SkillDao的代理对象
            SkillDao dao = session.getMapper(SkillDao.class);
            Skill skill = new Skill("666", 99);
            skill.setId(22);
            Assert.assertTrue(dao.update(skill));
        }
    }

    @Test
    public void remove() {
        try (SqlSession session = MyBatises.openSession(true)) {
            // 生成SkillDao的代理对象
            SkillDao dao = session.getMapper(SkillDao.class);
            Assert.assertTrue(dao.remove(22));
        }
    }
}
```

> **这里在获取到session后，直接用session的getMapper()方法加载xxxDao接口的代理对象；**

> **当执行到dao.get()或者list()、save()等方法时，会去namespace为com.ws.dao.SkillDao中选择对应的方法执行；**

> **mapper的namespace必须是dao接口类的全类名；**

> **mapper中的select、update、insert、delete的id值必须和dao的方法名是一致的；**

> **如果update、insert、delete方法的返回值是Boolean类型，则代理对象内部是影响记录数大于0就返回true；**


#### **3.只定义dao接口类，SqlSession的getMapper方法生成dao的代理对象（使用注解）**

> **同样不在dao中具体实现impl，也不编写映射文件xml，只是用注解的方式在dao接口类对应方法上注解编写sql语句，所以在mybatis配置文件中也不用实现实体映射**

**创建dao接口：并没有用xxxDaoImpl实现这个接口，也没有映射文件xml**
```java
@CacheNamespace(flushInterval = 600000, size = 512, readWrite = true)
public interface SkillDao {
//    @SelectKey(statement = "SELECT LAST_INSERT_ID()",
//            keyProperty = "id", before = false, resultType = Integer.class)

//    @Options(useGeneratedKeys = true, keyProperty = "id")

    @Insert("INSERT INTO skill(name, level) VALUES (#{name}, #{level})")
    boolean save(Skill skill);

    @Update("UPDATE skill SET name = #{name}, level = #{level} WHERE id = #{id}")
    boolean update(Skill skill);

    @Delete("DELETE FROM skill WHERE id = #{id}")
    boolean remove(Integer id);

    @Select("SELECT * FROM skill WHERE id = #{id}")
    Skill get(Integer id);

    @Select("SELECT * FROM skill")
    List<Skill> list();

    @Select("SELECT * FROM skill LIMIT #{start}, #{size}")
    //@Param:用来设置参数名，用来将穿进去的int start/size与sql语句的#{start}/#{size}对应
    List<Skill> listByStartAndSize(@Param("start") int start,
                                   @Param("size") int size);
}
```

#### **测试代码**

```java
public class SkillTest {
    @Test
    public void get() {
        try (SqlSession session = MyBatises.openSession()) {
            // 生成SkillDao的代理对象
            SkillDao dao = session.getMapper(SkillDao.class);
            System.out.println(dao.get(2));
        }
    }

    @Test
    public void list() {
        try (SqlSession session = MyBatises.openSession()) {
            // 生成SkillDao的代理对象
            SkillDao dao = session.getMapper(SkillDao.class);
            System.out.println(dao.list());
        }
    }

    @Test
    public void list2() {
        try (SqlSession session = MyBatises.openSession()) {
            // 生成SkillDao的代理对象
            SkillDao dao = session.getMapper(SkillDao.class);
            List<Skill> skills = dao.listByStartAndSize(0, 10);
            for (Skill skill : skills) {
                System.out.println(skill);
            }
        }
    }

    @Test
    public void save() {
        try (SqlSession session = MyBatises.openSession(true)) {
            // 生成SkillDao的代理对象
            SkillDao dao = session.getMapper(SkillDao.class);
            Assert.assertTrue(dao.save(new Skill("mj888", 100)));
        }
    }

    @Test
    public void update() {
        try (SqlSession session = MyBatises.openSession(true)) {
            // 生成SkillDao的代理对象
            SkillDao dao = session.getMapper(SkillDao.class);
            Skill skill = new Skill("666", 99);
            skill.setId(17);
            Assert.assertTrue(dao.update(skill));
        }
    }

    @Test
    public void remove() {
        try (SqlSession session = MyBatises.openSession(true)) {
            // 生成SkillDao的代理对象
            SkillDao dao = session.getMapper(SkillDao.class);
            Assert.assertTrue(dao.remove(17));
        }
    }
}
```

**注意：**
- 在上面的xxxDao接口代码中，使用到了注解的方式，除了常用的 `@select`、`@insert`、`@update`等，还有多表关联后的注解写法和标签对应，即纯注解与xml文件的标签对应关系：
- `@param` 设置参数名，以将传入参数与sql语句参数占位标志对应
- `@results、@resultMap` 对应映射文件中字段映射 `<resultMap>` 
- `@result` 对应 `<id>`、`<result>`
- `@one` 对应 `<association>` 单表
- `@many` 对应 `<collection>` 多表关联
- `@selectedKey` 返回操作的那个数据的id 对应之前的 `keyProperty` 与 `useGeneratedKeys` 两种写法 *
- `@CacheNamespace` 开启二级缓存，没有了xml文件用这个代替，其中属性 `readWrite` 对应之前的 `readOnly（序列化与反序列化）`

**使用MyBatis实现Dao层的几种方式总结：**
1. 第一种代码冗余，较差；
2. 目前来看注解比较简单，也没有映射文件，直接将sql语句注解在dao接口类中，不用具体实现；
3. 但缺点也有：①简单增删改查还行，当涉及多表关联查询时，sql语句十分复杂，注解没有xml文件逻辑清晰；②当把sql写在dao的接口类中后，java代码会编译成.class文件，不能在项目部署后直接修改，十分麻烦，即不能动态编辑sql语句，而xml文件.xml可以直接打开编辑；
4. 所以使用注解与xml文件的混用，如果sql语句较为复杂或者想动态修改的sql语句，我们可以写在xml文件中；


## **MyBatis的常见问题解答**

#### **1、#{}和\${}的区别是什么？**

答：

- `${}`是 Properties 文件中的变量占位符，它可以用于标签属性值和 sql 内部，属于静态文本替换，比如\${driver}会被静态替换为`com.mysql.jdbc.Driver`。
- `#{}`是 sql 的参数占位符，MyBatis 会将 sql 中的`#{}`替换为?号，在 sql 执行前会使用 PreparedStatement 的参数设置方法，按序给 sql 的?号占位符设置参数值，比如 ps.setInt(0, parameterValue)，`#{item.name}` 的取值方式为使用反射从参数对象中获取 item 对象的 name 属性值，相当于 `param.getItem().getName()`。

#### **2、Xml 映射文件中，除了常见的 select|insert|update|delete 标签之外，还有哪些标签？**

答：还有很多其他的标签，`<resultMap>`、`<parameterMap>`、`<sql>`、`<include>`、`<selectKey>`，加上动态 sql 的 9 个标签，`trim|where|set|foreach|if|choose|when|otherwise|bind`等，其中<sql>为 sql 片段标签，通过`<include>`标签引入 sql 片段，`<selectKey>`为不支持自增的主键生成策略标签。

#### **3、最佳实践中，通常一个 Xml 映射文件，都会写一个 Dao 接口与之对应，请问，这个 Dao 接口的工作原理是什么？Dao 接口里的方法，参数不同时，方法能重载吗？**

答：Dao 接口，就是人们常说的 `Mapper`接口，接口的全限名，就是映射文件中的 namespace 的值，接口的方法名，就是映射文件中`MappedStatement`的 id 值，接口方法内的参数，就是传递给 sql 的参数。`Mapper`接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为 key 值，可唯一定位一个`MappedStatement`，举例：`com.mybatis3.mappers.StudentDao.findStudentById`，可以唯一找到 namespace 为`com.mybatis3.mappers.StudentDao`下面`id = findStudentById`的`MappedStatement`。在 MyBatis 中，每一个`<select>`、`<insert>`、`<update>`、`<delete>`标签，都会被解析为一个`MappedStatement`对象。

Dao 接口里的方法可以重载，但是Mybatis的XML里面的ID不允许重复。

Mybatis版本3.3.0，亲测如下：

```java
/**
 * Mapper接口里面方法重载
 */
public interface StuMapper {

	List<Student> getAllStu();
    
	List<Student> getAllStu(@Param("id") Integer id);
}
```

然后在 `StuMapper.xml` 中利用Mybatis的动态sql就可以实现。

```java
	<select id="getAllStu" resultType="com.pojo.Student">
 		select * from student
		<where>
			<if test="id != null">
				id = #{id}
			</if>
		</where>
 	</select>
```

能正常运行，并能得到相应的结果，这样就实现了在Dao接口中写重载方法。

**Mybatis 的 Dao 接口可以有多个重载方法，但是多个接口对应的映射必须只有一个，否则启动会报错。**

相关 issue ：[更正：Dao 接口里的方法可以重载，但是Mybatis的XML里面的ID不允许重复！](https://github.com/Snailclimb/JavaGuide/issues/1122)。

Dao 接口的工作原理是 JDK 动态代理，MyBatis 运行时会使用 JDK 动态代理为 Dao 接口生成代理 proxy 对象，代理对象 proxy 会拦截接口方法，转而执行`MappedStatement`所代表的 sql，然后将 sql 执行结果返回。

##### ==补充：==

Dao接口方法可以重载，但是需要满足以下条件：

1. 仅有一个无参方法和一个有参方法
2. 多个有参方法时，参数数量必须一致。且使用相同的 `@Param` ，或者使用 `param1` 这种

测试如下：

`PersonDao.java`

```java
Person queryById();

Person queryById(@Param("id") Long id);

Person queryById(@Param("id") Long id, @Param("name") String name);
```

`PersonMapper.xml`

```xml
<select id="queryById" resultMap="PersonMap">
    select
      id, name, age, address
    from person
    <where>
        <if test="id != null">
            id = #{id}
        </if>
        <if test="name != null and name != ''">
            name = #{name}
        </if>
    </where>
    limit 1
</select>
```

`org.apache.ibatis.scripting.xmltags.DynamicContext.ContextAccessor#getProperty`方法用于获取`<if>`标签中的条件值

```java
public Object getProperty(Map context, Object target, Object name) {
  Map map = (Map) target;

  Object result = map.get(name);
  if (map.containsKey(name) || result != null) {
    return result;
  }

  Object parameterObject = map.get(PARAMETER_OBJECT_KEY);
  if (parameterObject instanceof Map) {
    return ((Map)parameterObject).get(name);
  }

  return null;
}
```

`parameterObject`为map，存放的是Dao接口中参数相关信息。

`((Map)parameterObject).get(name)`方法如下

```java
public V get(Object key) {
  if (!super.containsKey(key)) {
    throw new BindingException("Parameter '" + key + "' not found. Available parameters are " + keySet());
  }
  return super.get(key);
}
```

1. `queryById()`方法执行时，`parameterObject`为null，`getProperty`方法返回null值，`<if>`标签获取的所有条件值都为null，所有条件不成立，动态sql可以正常执行。
2. `queryById(1L)`方法执行时，`parameterObject`为map，包含了`id`和`param1`两个key值。当获取`<if>`标签中`name`的属性值时，进入`((Map)parameterObject).get(name)`方法中，map中key不包含`name`，所以抛出异常。
3. `queryById(1L,"1")`方法执行时，`parameterObject`中包含`id`,`param1`,`name`,`param2`四个key值，`id`和`name`属性都可以获取到，动态sql正常执行。

#### **4、MyBatis 是如何进行分页的？分页插件的原理是什么？**

答：**(1)** MyBatis 使用 RowBounds 对象进行分页，它是针对 ResultSet 结果集执行的内存分页，而非物理分页；**(2)** 可以在 sql 内直接书写带有物理分页的参数来完成物理分页功能，**(3)** 也可以使用分页插件来完成物理分页。

分页插件的基本原理是使用 MyBatis 提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的 sql，然后重写 sql，根据 dialect 方言，添加对应的物理分页语句和物理分页参数。

举例：`select _ from student`，拦截 sql 后重写为：`select t._ from （select \* from student）t limit 0，10`

#### **5、简述 MyBatis 的插件运行原理，以及如何编写一个插件。**

答：MyBatis 仅可以编写针对 `ParameterHandler`、`ResultSetHandler`、`StatementHandler`、`Executor` 这 4 种接口的插件，MyBatis 使用 JDK 的动态代理，为需要拦截的接口生成代理对象以实现接口方法拦截功能，每当执行这 4 种接口对象的方法时，就会进入拦截方法，具体就是 `InvocationHandler` 的 `invoke()`方法，当然，只会拦截那些你指定需要拦截的方法。

实现 MyBatis 的 Interceptor 接口并复写` intercept()`方法，然后在给插件编写注解，指定要拦截哪一个接口的哪些方法即可，记住，别忘了在配置文件中配置你编写的插件。

#### 6、**MyBatis 执行批量插入，能返回数据库主键列表吗？**

答：能，JDBC 都能，MyBatis 当然也能。

#### **7、MyBatis 动态 sql 是做什么的？都有哪些动态 sql？能简述一下动态 sql 的执行原理不？**


答：MyBatis 动态 sql 可以让我们在 Xml 映射文件内，以标签的形式编写动态 sql，完成逻辑判断和动态拼接 sql 的功能，MyBatis 提供了 9 种动态 sql 标签 `trim|where|set|foreach|if|choose|when|otherwise|bind`。

其执行原理为，使用 OGNL 从 sql 参数对象中计算表达式的值，根据表达式的值动态拼接 sql，以此来完成动态 sql 的功能。

#### **8、MyBatis 是如何将 sql 执行结果封装为目标对象并返回的？都有哪些映射形式？**

答：第一种是使用`<resultMap>`标签，逐一定义列名和对象属性名之间的映射关系。第二种是使用 sql 列的别名功能，将列别名书写为对象属性名，比如 T_NAME AS NAME，对象属性名一般是 name，小写，但是列名不区分大小写，MyBatis 会忽略列名大小写，智能找到与之对应对象属性名，你甚至可以写成 T_NAME AS NaMe，MyBatis 一样可以正常工作。

有了列名与属性名的映射关系后，MyBatis 通过反射创建对象，同时使用反射给对象的属性逐一赋值并返回，那些找不到映射关系的属性，是无法完成赋值的。

#### **9、MyBatis 能执行一对一、一对多的关联查询吗？都有哪些实现方式，以及它们之间的区别。**

答：能，MyBatis 不仅可以执行一对一、一对多的关联查询，还可以执行多对一，多对多的关联查询，多对一查询，其实就是一对一查询，只需要把 `selectOne()`修改为 `selectList()`即可；多对多查询，其实就是一对多查询，只需要把 `selectOne()`修改为 `selectList()`即可。

关联对象查询，有两种实现方式，一种是单独发送一个 sql 去查询关联对象，赋给主对象，然后返回主对象。另一种是使用嵌套查询，嵌套查询的含义为使用 join 查询，一部分列是 A 对象的属性值，另外一部分列是关联对象 B 的属性值，好处是只发一个 sql 查询，就可以把主对象和其关联对象查出来。

那么问题来了，join 查询出来 100 条记录，如何确定主对象是 5 个，而不是 100 个？其去重复的原理是`<resultMap>`标签内的`<id>`子标签，指定了唯一确定一条记录的 id 列，MyBatis 根据<id>列值来完成 100 条记录的去重复功能，`<id>`可以有多个，代表了联合主键的语意。

同样主对象的关联对象，也是根据这个原理去重复的，尽管一般情况下，只有主对象会有重复记录，关联对象一般不会重复。

举例：下面 join 查询出来 6 条记录，一、二列是 Teacher 对象列，第三列为 Student 对象列，MyBatis 去重复处理后，结果为 1 个老师 6 个学生，而不是 6 个老师 6 个学生。

| t_id | t_name  | s_id |
| ---- | ------- | ---- |
| 1    | teacher | 38   |
| 1    | teacher | 39   |
| 1    | teacher | 40   |
| 1    | teacher | 41   |
| 1    | teacher | 42   |
| 1    | teacher | 43   |

#### **10、MyBatis 是否支持延迟加载？如果支持，它的实现原理是什么？**

答：MyBatis 仅支持 association 关联对象和 collection 关联集合对象的延迟加载，association 指的就是一对一，collection 指的就是一对多查询。在 MyBatis 配置文件中，可以配置是否启用延迟加载 `lazyLoadingEnabled=true|false。`

它的原理是，使用` CGLIB` 创建目标对象的代理对象，当调用目标方法时，进入拦截器方法，比如调用 `a.getB().getName()`，拦截器 `invoke()`方法发现 `a.getB()`是 null 值，那么就会单独发送事先保存好的查询关联 B 对象的 sql，把 B 查询上来，然后调用 a.setB(b)，于是 a 的对象 b 属性就有值了，接着完成 `a.getB().getName()`方法的调用。这就是延迟加载的基本原理。

当然了，不光是 MyBatis，几乎所有的包括 Hibernate，支持延迟加载的原理都是一样的。

#### **11、MyBatis 的 Xml 映射文件中，不同的 Xml 映射文件，id 是否可以重复？**

答：不同的 Xml 映射文件，如果配置了 namespace，那么 id 可以重复；如果没有配置 namespace，那么 id 不能重复；毕竟 namespace 不是必须的，只是最佳实践而已。

原因就是 namespace+id 是作为 `Map<String, MappedStatement>`的 key 使用的，如果没有 namespace，就剩下 id，那么，id 重复会导致数据互相覆盖。有了 namespace，自然 id 就可以重复，namespace 不同，namespace+id 自然也就不同。

#### **12、MyBatis 中如何执行批处理？**

答：使用 BatchExecutor 完成批处理。

#### **13、MyBatis 都有哪些 Executor 执行器？它们之间的区别是什么？**

答：MyBatis 有三种基本的 Executor 执行器，**`SimpleExecutor`、`ReuseExecutor`、`BatchExecutor`。**

**`SimpleExecutor`：**每执行一次 update 或 select，就开启一个 Statement 对象，用完立刻关闭 Statement 对象。

**`ReuseExecutor`：**执行 update 或 select，以 sql 作为 key 查找 Statement 对象，存在就使用，不存在就创建，用完后，不关闭 Statement 对象，而是放置于 Map<String, Statement>内，供下一次使用。简言之，就是重复使用 Statement 对象。

**`BatchExecutor`：**执行 update（没有 select，JDBC 批处理不支持 select），将所有 sql 都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它缓存了多个 Statement 对象，每个 Statement 对象都是 addBatch()完毕后，等待逐一执行 executeBatch()批处理。与 JDBC 批处理相同。

作用范围：Executor 的这些特点，都严格限制在 SqlSession 生命周期范围内。

#### **14、MyBatis 中如何指定使用哪一种 Executor 执行器？**

答：在 MyBatis 配置文件中，可以指定默认的 ExecutorType 执行器类型，也可以手动给 `DefaultSqlSessionFactory` 的创建 SqlSession 的方法传递 ExecutorType 类型参数。

#### **15、MyBatis 是否可以映射 Enum 枚举类？**

答：MyBatis 可以映射枚举类，不单可以映射枚举类，MyBatis 可以映射任何对象到表的一列上。映射方式为自定义一个 `TypeHandler`，实现 `TypeHandler` 的 `setParameter()`和 `getResult()`接口方法。`TypeHandler` 有两个作用，一是完成从 javaType 至 jdbcType 的转换，二是完成 jdbcType 至 javaType 的转换，体现为 `setParameter()`和 `getResult()`两个方法，分别代表设置 sql 问号占位符参数和获取列查询结果。

#### **16、MyBatis 映射文件中，如果 A 标签通过 include 引用了 B 标签的内容，请问，B 标签能否定义在 A 标签的后面，还是说必须定义在 A 标签的前面？**

答：虽然 MyBatis 解析 Xml 映射文件是按照顺序解析的，但是，被引用的 B 标签依然可以定义在任何地方，MyBatis 都可以正确识别。

原理是，MyBatis 解析 A 标签，发现 A 标签引用了 B 标签，但是 B 标签尚未解析到，尚不存在，此时，MyBatis 会将 A 标签标记为未解析状态，然后继续解析余下的标签，包含 B 标签，待所有标签解析完毕，MyBatis 会重新解析那些被标记为未解析的标签，此时再解析 A 标签时，B 标签已经存在，A 标签也就可以正常解析完成了。

#### **17、简述 MyBatis 的 Xml 映射文件和 MyBatis 内部数据结构之间的映射关系？**

答：MyBatis 将所有 Xml 配置信息都封装到 All-In-One 重量级对象 Configuration 内部。在 Xml 映射文件中，`<parameterMap>`标签会被解析为 `ParameterMap` 对象，其每个子元素会被解析为 ParameterMapping 对象。`<resultMap>`标签会被解析为 `ResultMap` 对象，其每个子元素会被解析为 `ResultMapping` 对象。每一个`<select>、<insert>、<update>、<delete>`标签均会被解析为 `MappedStatement` 对象，标签内的 sql 会被解析为 BoundSql 对象。

#### **18、为什么说 MyBatis 是半自动 ORM 映射工具？它与全自动的区别在哪里？**
答：Hibernate 属于全自动 ORM 映射工具，使用 Hibernate 查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取，所以它是全自动的。而 MyBatis 在查询关联对象或关联集合对象时，需要手动编写 sql 来完成，所以，称之为半自动 ORM 映射工具。

面试题看似都很简单，但是想要能正确回答上来，必定是研究过源码且深入的人，而不是仅会使用的人或者用的很熟的人，以上所有面试题及其答案所涉及的内容，在我的 MyBatis 系列博客中都有详细讲解和原理分析。