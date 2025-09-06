## 前言

随着业务的发展，**用户对系统需求变得越来越多**，这就要求系统能够快速更新迭代以满足业务需求，通常系统版本发布时，都要**先执行数据库的DDL变更**，**包括创建表、添加字段、添加索引、修改字段属性等。**

在**数据量大不大的情况下，执行DDL都很快**，对业务基本没啥影响，**但是数据量大的情况，而且我们业务做了读写分离，接入了实时数仓，这时DDL变更就是一个的难题，需要综合各方业务全盘考虑。**

## MySQL中的DDL
### DDL概述

MySQL中的**DDL语句形式比较多**，概括一下有以下几类：**CREATE，ALTER，DROP，RENAME，TRUNCATE。**

这些操作都是隐式提交且原子性，要么成功，要么失败，在MySQL 8.0之前DDL操作是不记录日志的。

今天就聊一下跟系统版本发布相关的数据库结构变更，主要就是**ALTER TABLE**变更了，DDL变更流程普通的DML变更是类似的，如下所示

![](https://img-blog.csdnimg.cn/b0b5cf2cbf844910b1fc95af3b0d0b96.png)


**在早期的MySQL版本，DDL变更都会导致全表被锁，阻塞表上的DML操作，影响业务正常运行**，好的一点就是，随着MySQL版本的迭代，DDL的执行方式也在变化。

### MetaData元数据

**MySQL的元数据（MetaData）** 跟其他的RDBMS数据库一样的，**描述的对象的结构信息**，存储在`information_schema`架构下，例如常见的`TABLES`、`COLUMNS`等，下面例子是创建一个表crm_users，MySQL会自动往Information_schema.tables和columns等相关数据字典表中插入数据，这些数据称为元数据，一般都是静态化，只有表上发生了DDL操作才会实时更新。

![](https://img-blog.csdnimg.cn/a51d867bc567450da950847d1a18c668.png)

### MetaData Lock

**MySQL利用MetaData Lock来管理对象的访问**，**保证数据的一致性**，**对于一些核心业务表，表上DML操作比较频繁，这个时候添加字段可能会触发MetaData Lock。**

![](https://img-blog.csdnimg.cn/33aa6bdb96f342ca9519eb65cf43314a.png)

![](https://img-blog.csdnimg.cn/ca99c3c213d64065b3392e34be336879.png)

可以看到`Waiting for table metadata lock`等待事件，thread 155正在执行`alter table`等待thread 154执行的`select`释放锁，因为**DML在执行期间会持有SHARED_READ锁**，**要执行DDL时获取SHARED_UPGRADABLE（共享可升级锁，缩写为SU，允许并发更新和读同一个表）锁成功，但是获取EXCLUSIVE MetaData Lock锁失败，处于暂挂PENDING状态。**

## DDL执行方式

从MySQL官方文档可以看到，ALTER TABLE的选项很多，跟性能相关的选项主要有`ALGORITHM`和`LOCK`。
![](https://img-blog.csdnimg.cn/264837a3fdb24b3ba4e04c451e397a5e.png)

|ALGORITHM OPTION | DESCRIPTION|
|------- |-------|
|**COPY** | **MySQL早期的变更方式，需要创建修改后的临时表，然后按数据行拷贝原表数据到临时表，做rename重命名来完成创建，在此期间不允许并发DML操作，原表是可读的，不可写，同时需要额外一倍的磁盘空间。**|
|**INPLACE**|**直接在原表上进行修改，不需创建临时表拷贝数据及重命名，原表会持有Exclusive Metadata  Lock，通常是允许并发DML操作。**|
|**INSTANT**|MySQL 5.8开始支持，**只修改数据字典中的元数据，表数据不受影响，执行期间没有Exclusive Metadata  Lock，允许并发的DML操作。**|

从这张表可以看到，MySQL对于DDL执行方式一直在做优化，目的就是为了**提高DDL执行效率，减少锁等待，不影响表数据，同时不影响正常的DML操作。**

### LOCK选项

|LOCK OPTiON|	DESCRIPTION|
|---|---|
|DEFAULT	|默认模式：MySQL根据运行情况，在尽量不锁表的情况下自动选择LOCK模式。|
|NONE	|无锁：允许Online DDL期间进行并发读写操作，如果Online DDL操作不支持对表并发DML操作，则DDL操作失败，对表修改无效。|
|SHARED	|共享锁：Online DDL操作期间不影响读取，阻塞写入。|
|EXCLUSIVE|	排它锁：Online DDL操作期间不允许对锁表进行任何操作。|

### COPY

COPY方式的变更流程如下：

![](https://img-blog.csdnimg.cn/35025b2cab6c48458af4e7a803eef777.png)

根据业务需要，需要在crm_users添加一个字段user_type，采用COPY方式执行变更。

![](https://img-blog.csdnimg.cn/97b79b958d704a3c9b7fe215ebdaea8b.png)

![](https://img-blog.csdnimg.cn/fae5b4e37ae9473b917339f35bf9ebdb.png)

> **从执行过程及profile可以看出，通过COPY方式会创建临是表#sql-564_85，获取System Lock，拷贝数据到临时表，最后做rename表名切换，释放Lock资源，在执行期间不支持并发DML操作。**

### INPLACE

INPLACE方式是在原表上直接修改，对于添加索引、添加/删除列、修改字段NULL/NOT NULL属性等操作，需要修改MySQL内部的数据记录，需要重建表（Rebuild Table）。

![](https://img-blog.csdnimg.cn/25565313bfc9497599be9adc1763b7b1.png)

![](https://img-blog.csdnimg.cn/9ea1e67f49494200a56a1b1524232037.png)

![](https://img-blog.csdnimg.cn/0e6b2abcfc2c45aaac5cb2e34d41bd69.png)

> **从执行过程可以看到，需要获取Exclusive Metadata  Lock，修改表数据，释放Lock，在执行期间支持并发DML操作。**

### INSTANT

MySQL 5.8开始推出的方式，**DDL只修改数据字典中的元数据，表数据不受影响**，**没有Exclusive Metadata  Lock，允许并发的DML操作**，支持的DDL变更是有限制的，目前主要包括添加字段，添加/删除生成列，修改ENUM或SET列，改变索引类型以及重命名表。

![](https://img-blog.csdnimg.cn/7e212d79d9f14bd3991eeeffa371aa9e.png)

### 比对下这三种方式的执行效率

|执行方式/项目|	数据量(w)|	执行时间(s)|	重建表|	修改MetaData|	修改Data	|允许并发DML|
|---|---|---|---|---|---|--|
|COPY	|650|	29.89|	YES	|No|	Yes|	No|
|INPLACE|	650|	10.56	|No|	No|	Yes|	Yes|
|INSTANT|	650|	0.19|	No|	Yes|	No|	Yes|


### ONLINE DDL

截止MySQL 8.0，**OnLine DDL有三种方式COPY，INPLACE，INSTANT**，MySQL会自动根据执行的DDL选择使用哪种方式，一般会**优先选择INSTANT方式，如果不支持，就选择INPLANCE方式，再不支持就只能选择COPY方式了。**

## 大表DDL方案

在实际业务系统中，业务发展比较快，表的数据量比较大，业务层面又做了读写分离，同时会将MySQL数据实时同步到数据仓库（包括实时数仓和离线数仓），实际的数据库架构如下。

![](https://img-blog.csdnimg.cn/1a0ab2a7a15e414faf10aa6a79bae7cb.png)

> **假设这是一个交易系统数据库，订单表booking有8000w数据，且接入到了实时和离线仓库，根据业务需要，在订单表booking添加一个字段，在MySQL 5.7之前添加字段属于高危操作，需要充分考虑对业务的影响，主要存在于两个方面：**

- 在读写分离场景，主从同步延迟导致业务数据不一致
- 实时数仓ADB不允许源端MySQL表重命名，如果通过COPY方式或者pt-osc、gh-ost等工具都会rename表名，那么就需要从数仓删除该表，重新配置同步（全量 + 增量），会影响数仓业务


### 一、ONLINE DDL方式

对于MySQL 5.6到5.7的版本，可以使用OnLine DDL的方式变更，对于大表来说，执行时间会很长，**好处是在Master上DML操作不受影响，但是会导致主从延时。**

**假如Master上添加字段执行了20分钟，相应的Slave也要执行20分钟，在这期间Slave一直处于延迟状态，会造成业务数据不一致**，比如用户在Master下单成功，由于Slave延迟查询不到订单信息，用户误以为网络原因没有下单成功，又下了一单，导致重复下单的情况。

这种方式会导致**主从延迟，但是不会影响实时数仓的业务，根据业务情况，只能选择在业务低峰期执行了。**

### 二、pt-osc工具

为了解决DDL变更导致主从延时对业务的影响，会想到用大表变更利器pt-osc（pt-online-schema-change）或者gh-ost工具来做，这两个工具执行过程及原理大同小异，变更流程如下（不考虑外键，按照MySQL规范不允许使用外键）：

![](https://img-blog.csdnimg.cn/4f38ec78f4474482b61b80c785950d32.png)

1. 创建一个新的表，表结构为修改后的数据表，用于从源数据表向新表中导入数据。

2. 在源表上创建触发器，用于记录从拷贝数据开始之后，对源数据表继续进行数据修改的操作记录下来，用于数据拷贝结束后，执行这些操作，保证数据不会丢失。

3. 拷贝数据，从源数据表中拷贝数据到新表中。

4. 修改外键相关的子表，根据修改后的数据，修改外键关联的子表。

5. rename源数据表为old表，把新表rename为源表名，并将old表删除。

6. 删除触发器。

执行`pt-osc`的时候也需要获取一个`Exclusive Metadata  Lock`，**如果在此期间表上有DML操作正在进行，pt-osc操作会一直处于暂挂PENDING状态，这个时候表上正常DML操作都会被阻塞**，MySQL活动连接数瞬间暴涨，CPU使用率100%，依赖的该表的接口都会报错，所以要选择在业务**低峰期执行**，同时做好MetaData Lock锁的监控以便业务不受影响.

### 三、	MySQL 8.0变更方式

令人激动的是，MySQL 8.0也推出了INSTANT方式，**真正的只修改`MetaData`，不影响表数据，所以它的执行效率跟表大小几乎没有关系。**建议新系统上线用MySQL的话尽量使用MySQL 8.0，老的数据库也可以升级到MySQL 8.0获取更好的性能。

> **既要解决主从同步，又要解决rename数仓不同步的问题，目前只有INSTANT方式满足需求了。**

## 监控DDL执行进度

在大表执行DDL变更的时候，非常关心它的执行进度，MySQL 5.7之前是没有好的工具去监控，基本只能坐等了。在MySQL 8.0可以通过开启`performance_schema`，打开`events_stages_current`事件进行监控。

![](https://img-blog.csdnimg.cn/49e9e3b3e7c34b3082e0c8955bf733f0.png)

## 总结

DDL在业务系统版本迭代的过程是必不可少的，如何在不影响业务以及外围系统的情况下，实现DDL的平滑变更，是需要综合个系统特性考虑的，评估出重要性和优先级，同时也要掌握不同MySQL版本DDL执行方式，以便我们做更好的选择。

**例如上面提到了，目前在大数据团队，我们的业务都做了读写分离，同时接入实时数仓，数仓不支持rename操作，这时就可以选择在业务低峰期使用`ONLINE DDL`的方式执行，对业务系统影响最小，同时不影响数仓。**