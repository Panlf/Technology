# Flowable数据表介绍

- Flowable的所有数据库表都以ACT_开头。第二部分是说明表用途的两字符标示符。服务API的命名也大略符合这个规则。
- ACT_RE_: 'RE'代表repository。带有这个前缀的表包含“静态”信息，例如流程定义与流程资源（图片、规则等）。
- ACT_RU_: 'RU'代表runtime。这些表存储运行时信息，例如流程实例（process instance）、用户任务（user task）、变量（variable）、作业（job）等。Flowable只在流程实例运行中保存运行时数据，并在流程实例结束时删除记录。这样保证运行时表小和快。
- ACT_HI_: 'HI'代表history。这些表存储历史数据，例如已完成的流程实例、变量、任务等。
- ACT_GE_: 通用数据。在多处使用。

## 通用数据表

- act_ge_bytearray：二进制数据表，如流程定义、流程模板、流程图的字节流文件
- act_ge_property：属性数据表（不常用）

## 历史表（HistoryService接口操作的表）

- act_hi_actinst：历史节点表，存放流程实例运转的各个节点信息（包含开始、结束等非任务节点）
- act_hi_attachment：历史附件表，存放历史节点上传的附件信息（不常用）
- act_hi_comment：历史意见表
- act_hi_detail：历史详情表，存储节点运转的一些信息（不常用）
- act_hi_identitylink：历史流程人员表，存储流程各节点候选、办理人员信息，常用于查询某人或部门的已办任务
- act_hi_procinst：历史流程实例表，存储流程实例历史数据（包含正在运行的流程实例）
- act_hi_taskinst：历史流程任务表，存储历史任务节点
- act_hi_varinst：流程历史变量表，存储流程历史节点的变量信息

## 用户相关表（IdentityService接口操作的表）

- act_id_group：用户组信息表，对应节点选定候选组信息
- act_id_info：用户扩展信息表，存储用户扩展信息
- act_id_membership：用户与用户组关系表
- act_id_user：用户信息表，对应节点选定办理人或候选人信息


## 流程定义、流程模板相关表（RepositoryService接口操作的表）

- act_re_deployment：部属信息表，存储流程定义、模板部署信息
- act_re_procdef：流程定义信息表，存储流程定义相关描述信息，但其真正内容存储在act_ge_bytearray表中，以字节形式存储
- act_re_model：流程模板信息表，存储流程模板相关描述信息，但其真正内容存储在act_ge_bytearray表中，以字节形式存储

## 流程运行时表（RuntimeService接口操作的表）

- act_ru_task：运行时流程任务节点表，存储运行中流程的任务节点信息，重要，常用于查询人员或部门的待办任务时使用
- act_ru_event_subscr：监听信息表，不常用
- act_ru_execution：运行时流程执行实例表，记录运行中流程运行的各个分支信息（当没有子流程时，其数据与act_ru_task表数据是一一对应的）
- act_ru_identitylink：运行时流程人员表，重要，常用于查询人员或部门的待办任务时使用
- act_ru_job：运行时定时任务数据表，存储流程的定时任务信息
- act_ru_variable：运行时流程变量数据表，存储运行中的流程各节点的变量信息

