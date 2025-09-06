## 一、需求背景
- Y项目将要接入多种硬件设备，在开发、测试、生产几个不同的阶段，设备需要接入不同的软件环境，基于多租户的设备中心只能部署在单一环境中，无法解决设备在不同环境之间分配和切换的问题。物联网中台同样基于多租户权限系统，每个租户可以对设备进行分组管理，每个分组对应一种软件环境，物联网中台对外提供openAPI，部署在不同环境中的设备中心可以通过openAPI获取设备、进行设备相关的所有操作。
- 参考了阿里云、华为云等物联网平台产品，按照现阶段Y项目需求最小集的要求，引入了产品、设备、物模型、南向接口、北向接口、规则引擎等产品模块，可以实现物联网中台的必要功能，同时保留了扩展的可能性。
## 二、需求目标
- 多租户：利用现有多租户系统，后期可直接面向多个租户使用；
- 物模型、南向接口：制定统一的南向接口，新设备接入物联网中台无需开发，直接接入，物模型用来解析参数，相当于硬件协议，通过物模型中的产品模型解决级联设备的管理问题；
- 北向接口、设备分组：openAPI形式，支持跨域调用，可以面向多个环境，解决开发、测试、生产环境的设备分配问题；
- 规则引擎：物联网中台设置规则，生成事件或触发设备联动，由业务系统消费；
## 三、名词解释
- 产品：设备的集合，通常指一组具有相同功能的设备。
- 设备：归属于某个产品下的具体设备。
- 属性：设备可读取和设置的能力。一般用于描述设备运行时的状态，如环境监测设备所读取的当前环境温度等。属性支持GET和SET请求方式。应用系统可发起对属性的读取和设置请求。
- 事件：设备可被外部调用的能力或方法，可设置输入参数和输出参数。产品提供了什么功能供云端调用。相比于属性，服务可通过一条指令实现更复杂的业务逻辑，如执行某项特定的任务。
- 服务：设备运行时，主动上报给云端的事件。事件一般包含需要被外部感知和处理的通知信息，可包含多个输出参数。例如，某项任务完成的信息，或者设备发生故障或告警时的温度等，事件可以被订阅和推送。

## 四、需求分析
### 4.1 场景描述
- 通过描述还原参与场景的人、事、物。便于读者与产品统一对场景、痛点和目标的理解。
- 简单场景也可略过或通过流程图表达。

![](https://img-blog.csdnimg.cn/558a7064038e4817b3451feb56c8ab38.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbTBfNTEwNzAxODQ=,size_20,color_FFFFFF,t_70,g_se,x_16)

### 4.2 产品需求功能规划
- xx端＋xx模块＋xxx功能，可以使用脑图管理

![](https://img-blog.csdnimg.cn/e894d20c792e4f1caab242ff45d80584.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbTBfNTEwNzAxODQ=,size_20,color_FFFFFF,t_70,g_se,x_16)

### 4.3 需求列表

|端|	模块|	功能|	解决问题&目标|
|---|---|---|---|
|web端商户管理后台|登录	|登录租户 切换租户	||
||产品管理|	物模型	||
||设备管理-设备列表|设备列表|	显示和查询设备列表、设备统计|
|||设备分组	|分组对应不同的环境，利用分组实现设备分配到不同的环境|
|||设备详情|	显示基本信息 & 显示实时监测数据，查看历史监测数据列表 & 查看事件记录列表 & 查看服务调用记录列表|
|||子设备管理|显示一台设备的子设备列表 & 跳转至子设备详情页|
||设备管理-分组管理|分组列表||
|||创建分组	||
|||删除分组	||
|||分组设备列表	||
|||关联分组	||
|||取消分组	||
||规则引擎|规则列表|	显示和查询规则列表|
|||启用/停用|	设置规则的启用/停用状态|
|||新增|	新增规则|
|||详情	|查看和编辑规则详情 & 显示和查询规则执行记录列表|
|||删除	|删除规则|
||系统设置	|用户管理	|显示和查询用户列表 & 锁定/解锁用户 & 新建用户 & 查看和编辑用户详情 & 查看用户关联的角色列表，跳转至角色详情|
|||角色管理	|显示和查询角色列表 & 添加角色 & 查看和编辑角色信息、资源绑定 & 显示和查询用户列表、关联和取消关联用户 & 删除角色|
|||审计日志	|显示和查询审计日志|
||物模型	|产品	|设置产品信息|
|||属性	|设置设备属性|
|||事件	|设置设备事件|
|||服务	|设置设备服务|
||南向接口	|平台端	|设备属性值上报 & 设备事件上报 & 设备注册 & 设备批量注册 & 设备注销 & 设备启用 & 设备禁用|
|||设备端	|设备命令下发 & 查询设备列表 & 查询设备属性值、事件记录、服务调用记录 & 接入管理|
||北向接口|	产品管理	|查询产品列表 & 查询产品详细信息|
|||设备管理	|获取某个产品的设备列表 & 获取设备详情 & 编辑设备 & 锁定/解锁设备 & 获取设备数据、事件、服务指令下发记录 & 调用设备服务 & 设备事件推送 & 规则查询 & 规则创建 & 规则修改 & 接入管理|

## 五、详细需求文档

### 5.1 相关图示
#### 5.1.1 业务流程图

![](https://img-blog.csdnimg.cn/de1ee220cd0f4b849b706dbcbc1b52e3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbTBfNTEwNzAxODQ=,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 5.1.2 需求用例图
略！
#### 5.1.3 依赖关系图

![](https://img-blog.csdnimg.cn/95d75a8473864b6faa44a37d307c6b35.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbTBfNTEwNzAxODQ=,size_15,color_FFFFFF,t_70,g_se,x_16)

### 5.2 具体需求
略！

## 六、数据库设计

```sql
CREATE TABLE `iot_product_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `product_key` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '产品编码',
  `product_name` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '产品名称',
  `product_no` varchar(36) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '产品编码前缀',
  `node_type` varchar(16) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '设备类型 子设备  父级设备  非及联设备',
  `father_product_key` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '父级产品编码',
  `father_product_name` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '父级产品名称',
  `product_state` tinyint(1) NOT NULL DEFAULT '0' COMMENT '产品状态 0启用  1未启用',
  `product_cycle` int(11) NOT NULL DEFAULT '0' COMMENT '产品最小周期时间 单位S',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `version` varchar(16) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '1.0.0' COMMENT '版本号',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=25 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='产品信息表';





CREATE TABLE `iot_product_properties_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `product_key` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '产品编码',
  `product_name` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '产品名称',
  `identifier` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '产品属性code',
  `name` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '产品属性名称',
  `access_mode` varchar(16) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '读写模式，r只读,rw读写',
  `identifier_type` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '属性值数据类型',
  `specs` json NOT NULL COMMENT '属性具体json',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `product_id` bigint(20) NOT NULL COMMENT '关联产品表',
  `version` varchar(16) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '1.0.0' COMMENT '版本号',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=33 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='产品属性信息表';



CREATE TABLE `iot_product_events_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `product_key` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '产品编码',
  `product_name` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '产品名称',
  `identifier` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '产品事件code',
  `name` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '事件名称',
  `event_type` varchar(16) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '事件类型  信息info、告警alert、故障error',
  `output_data` json NOT NULL COMMENT '事件输出参数json',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `product_id` bigint(20) NOT NULL COMMENT '关联产品ID',
  `version` varchar(16) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '1.0.0' COMMENT '版本号',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=29 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='产品事件信息表';




CREATE TABLE `iot_product_services_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `product_key` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '产品编码',
  `product_name` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '产品名称',
  `identifier` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '服务code',
  `name` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '服务名称',
  `call_type` varchar(16) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '服务调用方式   async 异步  sync  同步 ',
  `service_method` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '调用方法',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `output_data` json NOT NULL COMMENT '服务输出参数json',
  `input_data` json NOT NULL COMMENT '服务输入参数json',
  `product_id` bigint(20) NOT NULL COMMENT '关联产品ID',
  `version` varchar(16) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '1.0.0' COMMENT '版本号',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='产品服务信息表';









CREATE TABLE `iot_device_info_record` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `device_sn_code` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '硬件ID',
  `device_name` varchar(36) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '设备名称',
  `device_code` varchar(36) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '设备编号',
  `device_type` varchar(10) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '设备类型 子设备  父级设备  非及联设备',
  `belong_to_project` varchar(18) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT 'unassigned' COMMENT '所属项目',
  `allocate_time` varchar(36) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '分配时间',
  `state` int(1) NOT NULL DEFAULT '1' COMMENT '0 在线  1 离线  在线状态',
  `version` varchar(10) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '版本号',
  `product_key` varchar(36) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '所属产品',
  `parent_product_code` varchar(36) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '上级产品编码',
  `parent_code` varchar(36) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '上级设备ID',
  `last_heart_time` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '最后在线时间:定时更新',
  `is_deleted` tinyint(1) NOT NULL DEFAULT '0' COMMENT '数据删除状态',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  `product_name` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '产品名称',
  `create_user` varchar(36) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '创建人',
  `enable_state` int(1) NOT NULL DEFAULT '0' COMMENT '启用状态  0 启用  1禁用',
  `fault_state` int(1) NOT NULL DEFAULT '0' COMMENT '故障状态  0 无故障  1故障',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=39 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='设备信息记录表';






CREATE TABLE `iot_device_online_state_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `device_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '设备ID',
  `monitoring_property` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '在线状态',
  `monitoring_property_name` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '在线状态名称',
  `online_state` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '在线状态',
  `last_heart_time` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '最后在线时间:定时更新',
  `active_time` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '激活时间',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=22 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='设备在线状态记录表';




CREATE TABLE `iot_monitoring_data_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `device_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '设备ID',
  `product_key` varchar(36) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '产品编码',
  `monitoring_property` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '监控属性',
  `monitoring_property_name` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '监控属性名称',
  `monitoring_value` varchar(1000) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '监控值',
  `monitoring_value_name` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '监控值',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `version_num` varchar(32) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '版本号',
  `unit` varchar(32) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '单位',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='设备监测数据记录表';







CREATE TABLE `iot_use_service_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `device_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '设备ID',
  `product_key` varchar(36) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '所属产品',
  `action_id` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '命令下发Id',
  `service_id` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '服务ID',
  `service_name` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '服务名称',
  `input_param` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '输入参数',
  `output_param` varchar(1000) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '输出参数',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `call_back_message` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '回调提示信息',
  `state` varchar(32) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT ' fail-失败  success-成功\n',
  `version_num` varchar(32) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '版本号',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=12 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='设备服务调用记录表';



CREATE TABLE `iot_event_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `device_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '设备ID',
  `product_key` varchar(36) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '所属产品',
  `event_id` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '事件ID',
  `event_name` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '事件名称',
  `event_type` varchar(32) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '事件类型',
  `event_type_name` varchar(32) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '事件类名称',
  `output_param` varchar(1000) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '输出参数',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `event_time` varchar(32) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '事件时间',
  `version_num` varchar(32) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '版本号',
  `message_id` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '上报事件ID',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='设备事件记录表';






CREATE TABLE `iot_rule_engine_record` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `rule_name` varchar(36) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '规则名称',
  `card_active` tinyint(2) NOT NULL DEFAULT '0' COMMENT '启用状态0 启用   1 禁用',
  `is_deleted` tinyint(1) NOT NULL DEFAULT '0' COMMENT '数据删除状态',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  `remark` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '备注',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='设备规则引擎表';





CREATE TABLE `iot_rule_engine_trigger_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `rule_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '规则ID',
  `version` varchar(10) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '版本号',
  `product_key` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '产品key',
  `trigger_type` varchar(16) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '触发类型   设备触发 时间触发',
  `trigger_mode` varchar(16) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '触发方式   属性触发  事件触发',
  `trigger_property` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '触发属性',
  `trigger_event` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '触发事件',
  `trigger_symbol` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '触发运算符',
  `trigger_value` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '触发值',
  `min_value` varchar(16) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '开始值',
  `max_value` varchar(16) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '结束值',
  `trigger_times` varchar(16) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '触发限制次数',
  `trigger_time_frame` varchar(16) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '触发时间限制 单位:分钟',
  `trigger_condition` varchar(16) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '触发条件',
  `trigger_repeat_mode` varchar(16) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '时间触发重复方式',
  `trigger_time` varchar(36) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '时间触发时刻',
  `trigger_week_time` varchar(36) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '时间触发每周时间',
  `trigger_month_time` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '时间触发月时间',
  `device_ids` varchar(36) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '时间触发选择设备集合',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='设备规则引擎触发器表';





CREATE TABLE `iot_rule_engine_execution_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `rule_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '规则ID',
  `execution_type` varchar(16) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '执行类型  设备动作、事件生成',
  `execution_service` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '执行服务',
  `execution_value` varchar(500) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '执行参数',
  `execution_delay_time` varchar(16) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '执行延时时间',
  `execution_mode` varchar(16) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '时间方式设备选择方式',
  `device_ids` varchar(500) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '时间触发选择设备集合',
  `execution_event` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '执行事件',
  `garden_name` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '设备所在职场名',
  `garden_id` int(11) NOT NULL DEFAULT '0' COMMENT '设备所在职场id',
  `building_name` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '设备所在楼栋名称',
  `building_id` int(11) NOT NULL DEFAULT '0' COMMENT '设备所在楼栋id',
  `floor_name` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '设备所在楼层名称',
  `floor_id` int(11) NOT NULL DEFAULT '0' COMMENT '设备所在楼层id',
  `area_name` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '设备所在区域名称',
  `area_id` int(11) NOT NULL DEFAULT '0' COMMENT '设备所在区域id',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='设备规则引擎执行器表';





CREATE TABLE `iot_rule_engine_execution_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `rule_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '规则ID',
  `execution_id` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '0' COMMENT '执行器ID',
  `trigger_id` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '0' COMMENT '触发器ID',
  `trigger_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '触发时间',
  `execution_action` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '执行动作',
  `trigger_mode` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '触发条件',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `execution_result` varchar(500) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '执行动作',
  `action_id` varchar(128) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '事件ID',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='设备规则引擎执行执行记录表';




CREATE TABLE `iot_service_event_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `identifier` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '业务事件',
  `identifier_name` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '业务事件名称',
  `output_data` json NOT NULL COMMENT '输出参数json',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='业务事件表';





CREATE TABLE `iot_user_op_record` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `tenant_code` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '租户编号',
  `user_name` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '用户名',
  `op_function` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '操作功能',
  `op_content` varchar(1500) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '操作内容',
  `op_ip` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '操作ip',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  `user_code` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '用户code',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='审计日志';





设备分组记录表  待定


CREATE TABLE `iot_device_group_info` (
	`id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键',
	`group_id` VARCHAR ( 64 ) NOT NULL DEFAULT '' COMMENT '分组ID',
	`group_name` VARCHAR ( 64 ) NOT NULL DEFAULT '' COMMENT '分组名称',
	`remark` VARCHAR ( 200 ) NOT NULL DEFAULT '' COMMENT '描述',
	`is_deleted` TINYINT ( 1 ) NOT NULL DEFAULT '0' COMMENT '数据删除状态',
	`update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
	`create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
PRIMARY KEY ( `id` ) 
) ENGINE = INNODB DEFAULT CHARSET = utf8mb4 COLLATE = utf8mb4_unicode_ci COMMENT = '设备分组表';





CREATE TABLE `iot_device_group_middle_info` (
	`id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键',
	`group_id` VARCHAR ( 64 ) NOT NULL DEFAULT '' COMMENT '分组ID',
	`device_id` VARCHAR ( 64 ) NOT NULL DEFAULT '' COMMENT '设备ID',
PRIMARY KEY ( `id` ) 
) ENGINE = INNODB DEFAULT CHARSET = utf8mb4 COLLATE = utf8mb4_unicode_ci COMMENT = '设备分组中间表';
```

## 七、表关系

![](https://img-blog.csdnimg.cn/fbb172a457c4470e85441db46f27b9b5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbTBfNTEwNzAxODQ=,size_20,color_FFFFFF,t_70,g_se,x_16)

## 八、接口设计

### 8.1 iot本地接口

|id|	接口名称|	path	|接口类型|	
|--|--|---|--|
|1	|设备列表-设备管理列表|/api/device/queryList|get	|
|2	|设备列表-产品选择下拉|	/api/product/list|	get|
|3|	设备列表-设备所属项目下拉|	/api/device/projectList|get|	
|4	|设备列表-设备管理列表统计设备数量接口|	/api/device/countList|	get	|
|5	|设备列表-设备状态下拉|	/api/device/stateList|get|	
|6	| 设备列表-启用禁用设备接口|	/api/device/updateState|put|	
|7	|设备列表-返回列表展示字段接口|	/api/device/fieldList|	get	|
|8	|设备列表-设备管理子设备列表|	/api/device/childList|	get	|
|11	|设备列表-批量分配项目接口|	/api/device/projectBelong|	put|	
|12	|设备列表-批量取消分配项目接口|	/api/device/cancelProjectBelong|put	|
|13|	设备详情-基本信息接口|	/api/device/basicInfo|	get|	
|14|	设备详情-监测数据接口|	/api/device/monitoringDataInfo|	get|	
|15|	设备详情-服务调用分页接口|	/api/device/serviceQueryList|	get	|
|16	|设备详情-事件管理分页接口|	/api/device/eventList|	get	|
|17	|设备详情-属性下拉接口|	/api/product/propertyList|	get	|
|18	|设备详情-事件类型	|/ap/product/eventTypeList|get	|
|19|	设备详情-事件名称下拉|	/api/product/eventNameLIst|	get|	
|20	|设备详情-服务下拉|	/api/product/serviceList|	get|	
|20|	规则引擎-规则列表分页接口|	/api/ruleEngine/queryList|	get	|
|21|	规则引擎-规则列表-批量删除接口|	/api/ruleEngine/deleteBatch|	delete	|
|22|	规则引擎-规则列表-批量启用停用接口|	/api/ruleEngine/updateStateBatch|	put|	
|22	|规则引擎-新增规则/编辑规则接口 |	/api/ruleEngine/submit|	post|	
|23|	规则引擎-详情接口|	/api/ruleEngine/detail|	get	|
|24|	规则引擎-触发器类型下拉|	/api/ruleEngine/triggerTypeList|	get|	
|25|	规则引擎-触发方式下拉|	/api/ruleEngine/triggerModeList|	get|	
|26|	规则引擎-属性下拉|	/api/ruleEngine/propertyList|	get	|
|27|	规则引擎-设备属性状态下拉|	/api/ruleEngine/propertyStateList|	get	|
|28|	规则引擎-重复方式下拉|	/api/ruleEngine/repetition Mode	|get	|
|29|	规则引擎-执行类型下拉|	/api/ruleEngine/executeType|	get	|
|30|	规则引擎-服务下拉|	/api/ruleEngine/serviceList|	get	|
|31|	规则引擎-空间树形接口|	/api/ruleEngine/workSpqceTree|	get	|
|32 |	执行记录列表|	/api/ruleEngine/executionRecordList|	get|	
|33|	规则引擎触发执行逻辑|||	
|33|	分组列表分页|	/api/group/queryList|	get	|
|34|	编辑/创建分组|	/api/group/submit|	post|	
|35|	删除分组|	/api/group/delete|	delete	|
|36|	批量删除分组|	/api/group/delete-batch	|delete	|
|37|	分组设备列表	|/api/group/deviceList|	get	|
|38	|关联设备|	/api/group/deviceBygroup|	put|	
|39	|取消关联设备|	/api/group/cancel-deviceBygroup	|put	|

### 8.2 北向接口

|id	|接口名称|	path|	接口类型|
|--|---|---|---|
|1	|获取某个产品在某个项目下的设备列表分页|/api/northward/productDeviceList|get|
|2|	获取设备详情|	/api/northward/deviceDetail|	get|
|3|	编辑设备|	/api/northward/deviceUpdate|	put|
|4|	设备启用	|/api/northward/deviceStart|	put|
|5|	设备禁用|	/api/northward/deviceCancel|	put|
|6|	获取设备数据历史记录分页|	/api/northward/deviceLnstructSend|	get|
|7|	获取服务指令下发记录分页|	/api/northward/deviceLnstructSendList	|get|
|8|	调用设备服务|	/api/northward/deviceUse|	post|
|9|	获取物模型|	/api/northward/thingList|	get|
|10|	设备事件推送|	/api/northward/deviceActionSend	|post|
|11|	设备属性值推送|	/api/northward/deviceProperty Send|	post|
|12|	获取设备事件历史记录|	/api/northward/deviceHistoryList|get|

### 8.3 南向接口

|id|	接口名称|	path|接口类型|
|---|---|---|---|
|1	|设备属性值上报|||
|2	|设备事件上报	|||
|3	|设备最新状态上报	|||
|4	|设备注册	|||
|5	|设备批量注册	|||
|6	|设备注销	|||
|7	|设备启用	|||
|8	|设备禁用	|||
|9	|设备命令下发||	|
|10	|查询设备列表	|||
|11	|查询设备最新状态	|||
|12	|查询设备最新属性值	|||
|13	|模拟南向数据	|||
|14	|绑定上级设备	|||
|15	|设备命令执行结果回调接口	|||
|16	|设备心跳上报	|||



## 九、物模型
### 9.1 数据类型
#### (1) int32
```java
"dataType": {
        "type": "int",
        "specs": {
          "min": "3",
          "max": "10",
          "unit": "mm/hour",
          "unitName": "降雨量",
          "step": "2"
        }
      }
```
#### (2) float
```java
"dataType": {
        "type": "float",
        "specs": {
          "min": "-3.1",
          "max": "6",
          "unit": "dS/m",
          "unitName": "土壤EC值",
          "step": "1.3"
        }
      }
```
#### (3) double
```java
"dataType": {
        "type": "double",
        "specs": {
          "min": "4.3",
          "max": "75",
          "unit": "r/min",
          "unitName": "转每分钟",
          "step": "1.65"
        }
      }
```
#### (4) enum
```java
"dataType": {
        "type": "enum",
        "specs": {
          "0": "在线",
          "1": "离线"
        }
      }
```
#### (5) bool
```java
"dataType": {
        "type": "bool",
        "specs": {
          "0": "关",
          "1": "开"
        }
      }
```
#### (6) text
```java
"dataType": {             //属性值数据类型信息
        "type": "text",          //属性值数据类型
        "specs": {               //取值范围
          "length": "62"       //最大字符数
        }
      }
```

#### (7) date
```java
"dataType": {
        "type": "date",
        "specs": {}
      }
```
#### (8) struct

**允许数据类型（每个参数有各自的数据类型，可以不同）：int32, float, double, enum, bool, text, date**
```java
"dataType": {
        "type": "struct",
        "specs": [
          {
            "identifier": "a",
            "name": "a",
            "dataType": {
              "type": "int",
              "specs": {
                "min": "4",
                "max": "8",
                "unit": "NTU",
                "unitName": "浊度",
                "step": "2"
              }
            }
          },
          {
            "identifier": "b",
            "name": "b",
            "dataType": {
              "type": "double",
              "specs": {
                "min": "5",
                "max": "8.6",
                "unit": "mg/kg",
                "unitName": "毫克每千克",
                "step": "1.4"
              }
            }
          }
        ]
      }
```
#### (9) array

**允许数据类型：int32, float, double, text, struct**
```java
示例1
"dataType": {
        "type": "array",
        "specs": {
          "size": "10",
          "item": {
            "type": "int"
          }
        }
      }
示例2
"dataType": {
            "type": "array",
            "specs": {
              "size": "10",
              "item": {
                "type": "struct",
                "specs": [
                  {
                    "identifier": "a",
                    "name": "a",
                    "dataType": {
                      "type": "int",
                      "specs": {
                        "min": "3",
                        "max": "89",
                        "unit": "var",
                        "unitName": "乏",
                        "step": "3"
                      }
                    }
                  },
                  {
                    "identifier": "b",
                    "name": "b",
                    "dataType": {
                      "type": "float",
                      "specs": {
                        "min": "3.6",
                        "max": "89.7",
                        "unit": "dS/m",
                        "unitName": "土壤EC值",
                        "step": "5.6"
                      }
                    }
                  }
                ]
              }
            }
```
### 9.2 搜狗智能固资柜主柜
```java
{
  "product": { //产品模块
    "version": "1.0.0",  //物模型版本号
    "productKey": "sogouAssetMajorCupboard"   //产品编码
    "productName": "搜狗智能固资柜主柜"      //产品名称
    "productNo": "0001"         //产品编号，用来自动生成设备ID
    "nodeType": "父节点"      //产品节点类型，影响产品列表、产品详情页个别字段
    "fatherProductKey": ""     //当前产品的父产品编码
    "fatherProductName": ""     //当前产品的父产品名称
    "productState": "0"、"1"     //产品发布状态，0代表开发中，1代表已发布，已发布产品不允许修改物模型
  },
  "properties":  [  //属性
    {
      "identifier": "firmwareVer",      //属性编码
      "name": "固件版本号",      //属性名称
      "accessMode": "r",        //读写模式，r只读,rw读写
      "dataType": {             //属性值数据类型信息
        "type": "text",          //属性值数据类型
        "specs": {               //取值范围
          "length": "62"       //最大字符数
        }
      }
    },
    {
      "identifier": "onlineState",
      "name": "在线状态",
      "accessMode": "r",
      "dataType": {
        "type": "enum",
        "specs": {
          "0": "在线",
          "1": "离线"
        }
      }
    }
  ],
  "events": [
  ],
  "services": [
  ]
}
```

### 9.3 搜狗智能固资柜副柜(14门)
```java
{
  "product": {
    "version": "1.0.0",
    "productKey": "sogouAssetMinorCupboard"
    "productName": "搜狗智能固资柜副柜(14门)"
    "productNo": "0002"
    "nodeType": "子节点"
    "fatherProductKey": "sogouAssetMajorCupboard"
    "fatherProductName": "搜狗智能固资柜主柜"
    "productState": "0"、"1"
  },
  "properties": [
    {
      "identifier": "onlineState",
      "name": "在线状态",
      "accessMode": "r",
      "dataType": {
        "type": "enum",
        "specs": {
          "0": "在线",
          "1": "离线"
        }
      }
    },
    {
      "identifier": "dooropenState",
      "name": "柜门开关状态",
      "accessMode": "r",
      "dataType": {
        "type": "struct",
        "specs": [
          {
            "identifier": "doorNum",
            "name": "柜门编号",
            "dataType": {
              "type": "int",
              "specs": {
                "min": "1",
                "max": "14",
                "unit": "",
                "unitName": "",
                "step": "1"
              }
            }
          },
          {
            "identifier": "state",
            "name": "状态",
            "dataType": {
              "type": "enum",
              "specs": {
                "0": "关闭",
                "1": "打开"
              }
            }
          }
        ]
      }
    }
  ],
  "events": [    
  ],
  "services": [           //服务
    {
      "identifier": "door1open",          //服务编码
      "name": "柜门1开门",             //服务名称
      "callType": "async",             //服务调用方式：异步
      "method": "thing.service.door1open",        //调用方法
      "inputData": [       //输入参数，数组，可为空
       {
          "identifier": "doorNum",
          "name": "柜门编号",
          "dataType": {
            "type": "int",
              "specs": {
              "min": "1",
              "max": "14",
              "unit": "",
              "unitName": "",
              "step": "1"
            }
          }
        }
      ],                    
      "outputData": [                   //输出参数
        {
          "identifier": "issueResult",       //参数编码
          "name": "指令下发状态",        //参数名称
          "dataType": {                   //参数数据类型信息
            "type": "enum",             //参数数据类型
            "specs": {                     //取值范围
              "0": "失败",                 //枚举值key-value
              "1": "成功"                  //枚举值key-value
            }
          }
        }
      ]
    }
  ]
}
```
### 9.4 搜狗门磁
```java
{
  "product": {
    "version": "1.0.0",
    "productKey": "sogouDoorSensor"
    "productName": "搜狗门磁"
    "productNo": "0003"
    "nodeType": "非级联设备"
    "fatherProductKey": ""
    "fatherProductName": ""
    "productState": "0"、"1"
},
  "properties": [
   {
      "identifier": "onlineState",
      "name": "在线状态",
      "accessMode": "r",
      "dataType": {
        "type": "enum",
        "specs": {
          "0": "在线",
          "1": "离线"
        }
      }
    },
    {
      "identifier": "openState",
      "name": "开关状态",
      "accessMode": "r",
      "dataType": {
        "type": "enum",
        "specs": {
          "0": "关闭",
          "1": "打开"
        }
      }
    },
    {
      "identifier": "batteryQua",
      "name": "电池电量",
      "accessMode": "r",
      "dataType": {
        "type": "int",
        "specs": {
          "min": "0",
          "max": "100",
          "unit": "%",
          "unitName": "百分比",
          "step": "1"
        }
      }
    }  
  ],
  "events": [
  ],
  "services": [
   ]
}
```
### 9.5 IT访客机
```java
{
  "product": {
    "version": "1.0.0",
    "productKey": "itVistorMachine"
    "productName": "IT访客机"
    "productNo": "0004"
    "nodeType": "非级联设备"
    "fatherProductKey": ""
    "fatherProductName": ""
    "productState": "0"、"1"
},
  "properties": [
    {
      "identifier": "state",
      "name": "状态",
      "accessMode": "r",
      "dataType": {
        "type": "struct",
        "specs": [
        {
          "identifier": "onlineState",
          "name": "在线状态",
          "dataType": {
            "type": "enum",
            "specs": {
              "0": "在线",
              "1": "离线"
            }
          }
        },
        {
          "identifier": "printerOnlineState",
          "name": "打印机连接状态",
          "dataType": {
            "type": "enum",
            "specs": {
              "0": "未连接",
              "1": "已连接"
            }
          }
        },
        {
          "identifier": "devState",
          "name": "设备状态",
          "dataType": {
            "type": "enum",
            "specs": {
              "1": "上线使用中",
              "2": "上线故障",
              "3": "下线"
            }
          }
        },
        {
          "identifier": "firmwareVer",
          "name": "固件版本号",
          "dataType": {
            "type": "text",          //属性值数据类型
            "specs": {               //取值范围
              "length": "62"
            }
          }
        },
        {
          "identifier": "lastHeartbeat",
          "name": "心跳时间",
          "dataType": {
            "type": "date",
            "specs": {               
            }
          }
        }
      ]
    }
  ],
  "events": [
    {
      "identifier": "heartbeatBreak",
      "name": "心跳中断",
      "type": "error",             //事件类型，详见prd
      "outputData": []
    },
    {
      "identifier": "printerOffline",
      "name": "打印机连接断开",
      "type": "error",
      "outputData": []
    },
    {
      "identifier": "devError",
      "name": "设备故障",
      "type": "error",
      "outputData": [
        {
          "identifier": "errorCode",       //参数编码
          "name": "故障码",        //参数名称
          "dataType": {                   //参数数据类型信息
            "type": "enum",             //参数数据类型
            "specs": {                     //取值范围
              "0": "恢复正常",                 //枚举值key-value
              "1": "打印机连接断开"                  //枚举值key-value
              "2": "打印机缺墨"                  //枚举值key-value
            }
          }
        }
      ]
    },

    {
      "identifier": "testHeartbeatBreak",
      "name": "测试心跳中断",
      "type": "info",             //事件类型，详见prd
      "outputData": [
        {
          "identifier": "lastHeartbeat1",
          "name": "心跳时间1",
          "dataType": {
            "type": "date",
            "specs": {               
            }
          }
        },
        {
          "identifier": "lastHeartbeat2",
          "name": "心跳时间2",
          "dataType": {
            "type": "date",
            "specs": {               
            }
          }
        }
      ]
    },
    {
      "identifier": "testPrinterOffline",
      "name": "测试打印机连接断开",
      "type": "alert",
      "outputData": []
    }
  ],
  "services": [
  ]
}
```