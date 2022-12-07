# Activiti概念

> 版本 7.0

## 入门概念
### 工作流

工作的一个流程，事物发展的一个业务过程

流程
	请假流程	员工-部门经理-总经理-人事存档

在计算机的帮助下，能够实现流程的自动化控制，就称为工作流。

### 工作流引擎

为了实现自动化控制，Activiti引擎就产生了

作用	实现流程自动化控制

### 工作流系统
具有工作流系统，如果一个系统具备流程的自动化管理功能，这个系统就可以称为工作流系统

硬代码问题	业务流程变更后，程序不能使用
	
Activiti可以实现业务流程变化后，程序代码不需要改动，只需要更新业务流程图。

### 使用场景
Saas 人力资源管理系统 	行政审批

### 原理

一个节点转换为一行数据

1、先将流程图画好
2、将流程图中每个节点的数据读取并放入表中
3、读取表中的第一条记录，处理并删除
	
实现这个自动化
```
1、业务流程图要规范化
2、这个业务流程图本质上是一个xml文件，这样就可以存入所要的数据
3、读取业务流程图的过程就是解析xml文件的过程
4、读取一个业务流程图中的节点相当于解析一个xml结构，进一步将数据插入到mysql表中，形成一条记录
5、将所有的节点都读取并存入mysql表中
6、后面只要读取mysql表中的记录就可以了，读一条记录就相当于读一个节点
7、业务流程的推进，后面就转化为读表中的数据，并且处理数据，结束时这一行数据就可以删除
```

### BPM

业务流程管理。

### BPMN

被各BPM厂商广泛接受的BPM标准。Activiti就是使用BPMN2.0进行流程建模、流程执行管理


### Saas-IHRM

1、整合Activiti
2、实现业务流程建模，使用BPMN实现业务流程图
3、部署业务流程到Activiti
4、启动流程实例
5、查询待办业务
6、处理待办业务
7、结束流程

### IDE安装插件

Eclipse 安装 activiti - https://www.activiti.org/designer/update/

Idea - settings-plugins-actiBPM

使用activiti可以实现业务代码不变更，就可以实现流程更新

流程定义图做更新 - 先读取节点信息到数据库表中 - 后续就针对数据库进行操作

## 数据表
```
act_re_*	re表示repository 	包含了流程定义和流程静态资源

act_ru_*	ru 表示Runtime	这些运行时的表，包含流程实例，任务，变量，异步任务等运行中数据。只在流程实例执行过程中保存这些数据，在流程结束时就会删除这些记录。这样运行时可以很小很快

act_hi_*	hi表示history	这些表包含历史数据，比如历史流程实例，变量，任务等

act_ge_*	ge表示general。通用数据，用于不同场景下。
```

##  activiti服务架构
```
	ProcessEngineConfiguration	《--- activiti.cfg.xml
	
			|
			|
			|
		ProcessEngine
			|
	
	RepositoryService 	TaskService 	IdentityService		FormService
		RuntimeService	ManagementService	HistoryService

activiti.cfg.xml

单机启动
org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration

通过org.activiti.spring.SpringProcessEngineConfiguration与Spring整合

RepositoryService	activiti的资源管理类
RuntimeService	activiti的流程运行管理类
TaskService		activiti的任务管理类
HistoryService	activiti的历史管理类
ManagerService	activiti的引擎管理类

Connection	连接
Event 事件
Task 任务
Gateway 网关
Container	容器
Boundary event	边界事件
Intermediate event 中间事件
```

## 开发流程
```
流程定义					
	bpmn	
流程部署
    activiti的三张表

流程实例
	流程定义好比是java的一个类
	流程实例好比是java中的一个实例对象，一个流程定义可以对应多个流程实例

1、ProcessEngineConfiguration类主要加载activiti.cfg.xml配置文件
	ProcessEngine类 作用是帮助我们可以快速得到各个Service接口，并且生成activiti工作环境	25张表生成
	Service接口	作用可以快速实现数据库25张表的操作

整个操作
1、画出流程定义图
2、部署流程定义
	方式一：单个文件 bpmn文件	png文件
	方式二：先将bpmn文件，png文件压缩成zip文件。
3、启动流程实例
	RuntimeService
		startProcessInstanceByKey("key")
4、查看任务
	TaskService			taskService.createTaskQuery()
5、完成任务
	TaskService			taskService.complete(task.getId());	//参数为任务ID

Businesskey	业务标识，通常为业务表的主键，业务标识和流程实例一一对应。业务标识来源于业务系统。
	存储业务标识就是根据业务标识来关联查询业务系统的数据。

个人任务
	固定分配	直接在插件中定死Assignee
	表达式		Assignee中写${user.assignee}、${assignee}
	监听器分配	继承TaskListener

流程变量
	比如请假天数不同走不同的流程

	如果将pojo存储到流程变量中，必须实现序列化接口Serializable，为了防止由于新增字段无法反序列化，
	需要生成serialVersionUID。
	
	流程变量的作用域
		一个流程实例、一个任务、一个执行实例，三个可以称为global变量
		一个任务、一个执行实例范围，范围没有流程实例大，称为local变量

	流程变量使用
		在连线上使用 - ${holidayNum>3}、${holidayNum<3}
	
	注意事项
		1、如果UEL表达式中流程变量名不存在则报错
		2、如果UEL表达式中流程变量为空NULL，流程不按UEL表达式去执行，而流程结束
		3、如果UEL表达式都不符合条件，流程结束
		4、如果不设置连线条件，会走flow序号小的那条线

组任务
Candidate Users 候选人
activiti:candidateUser="用户1,用户2,用户3"	设置一组候选人

第一步	查询组任务
第二步	拾取任务
		该组任务的所有候选人都能拾取

第三步	查询个人任务
第四部	办理个人任务

网关
	排他网关
		XOR(异或)网关
		用来在流程中实现决策。当流程执行到这个网关，所有分支都会判断条件是否为true，如果为true则执行该分支。
		注意，排他网关只会选择一个为true的分支执行(即使有两个分支都为true，也只会选择一条分支执行)
		ExclusiveGateway
	并行网关
		并行网关允许将流程分成多条分支，也可以将多条分支汇聚到一起，汇聚完成，并行网关结束
		fork分支

		join汇聚

		ParallelGateWay
		
		所有分支到达汇聚点，并行网关执行完成
		
	包含网关
		包含网关是可以看做排他网关和并行网关的结合体。
		
		InclusiveGateWay

Assignee	直接设置任务执行人
Candidate-user	设置候选用户，格式	wangwu,zhangsan,lisi
				依然需要指定具体的用户信息
Candidate-groups	特点不需要知道具体用户信息，只需要知道组名
```

## 表结构操作情况

```
1、资源库流程规则表
'RE'表示repository。 这个前缀的表包含了流程定义和流程静态资源 （图片，规则，等等）。
  1).act_re_deployment     部署信息表
  2).act_re_model      流程设计模型部署表
  3).act_re_procdef      流程定义数据表
2、运行时数据库表
'RU'表示runtime。 这些运行时的表，包含流程实例，任务，变量，异步任务，等运行中的数据。（ Activiti只在流程实例执行过程中保存这些数据， 在流程结束时就会删除这些记录。 这样运行时表可以一直很小速度很快。）
  1).act_ru_execution            运行时流程执行实例表
  2).act_ru_identitylink    运行时流程人员表，主要存储任务节点与参与者的相关信息
  3).act_ru_task        运行时任务节点表
  4).act_ru_variable        运行时流程变量数据表
3、历史数据库表
'HI'表示history。 这些表包含历史数据，比如历史流程实例， 变量，任务等等。
  1).act_hi_actinst         历史节点表
  2).act_hi_attachment        历史附件表
  3).act_hi_comment        历史意见表
  4).act_hi_identitylink    历史流程人员表
  5).act_hi_detail        历史详情表，提供历史变量的查询
  6).act_hi_procinst        历史流程实例表
  7).act_hi_taskinst        历史任务实例表
  8).act_hi_varinst        历史变量表
4、组织机构表
 'ID'表示identity。 这些表包含身份信息，比如用户，组等等。
  1).act_id_group        用户组信息表
  2).act_id_info        用户扩展信息表
  3).act_id_membership            用户与用户组对应信息表
  4).act_id_user        用户信息表
这四张表很常见，基本的组织机构管理，关于用户认证方面建议还是自己开发一套，组件自带的功能太简单，使用中有很多需求难以满足
5、通用数据表
'GE'表示general。通用数据， 用于不同场景下，如存放资源文件。
  1).act_ge_bytearray        二进制数据表
  2).act_ge_property        属性数据表存储整个流程引擎级别的数据,初始化表结构时，会默认插入三条记录
```