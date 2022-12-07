# Flowable介绍

## 介绍

Flowable是BPMN的一个基于java的软件实现，不过Flowable不仅仅包括BPMN，还有DMN决策表和CMMN Case管理引擎，并且有自己的用户管理、微服务API等一系列功能，是一个服务平台。

## Flowable UI

Flowable提供了几个web应用，用于演示及介绍Flowable项目提供的功能

- Flowable IDM 身份管理应用。为所有Flowable UI应用提供单点登录认证功能，并且为拥有IDM管理员权限的用户提供了管理用户、组和权限的功能。
- Flowable Modeler 让具有建模权限的用户可以创建流程模型、表单、选择表与应用定义
- Flowable Task 运行时任务应用。提供了启动流程实例、编辑任务表单、完成任务，以及查询流程实例与任务的功能。
- Flowable Admin 管理应用。让具有管理员权限的用户可以查询BPMN、DMN、Form及Content引擎，并提供了许多选项用于修改流程实例、任务、作业等。管理应用通过REST API连接至引擎，并与Flowable Task应用及Flowable REST应用一同部署。

所有其他的应用都需要Flowable IDM提供认证。每个应用的WAR文件可以部署在相同的servlet容器，也可以部署在不同的容器中。由于每个应用使用相同的cookie进行认证，因此应用需要运行在相同的域名下。

