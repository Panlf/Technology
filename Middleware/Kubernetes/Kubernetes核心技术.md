# Kubernetes核心技术

## 命令行工具kubectl

### 概述

kubectl是Kubernetes集群的命令行工具，通过kubectl能够对集群本身进行管理并能够在集群上进行容器化应用的暗转部署

### 语法

```
kubectl [command] [TYPE] [NAME] [flags]
```

- command 指定要对资源执行的操作，例如create get describe delete
- TYPE 指定资源类型，资源类型是大小写敏感，开发者能够以单数、复数和缩略的形式。

- NAME 指定资源的名称，名称也大小写敏感。如果省略名称，则会显示所有的资源。
- flags 指定可选参数。 例如 可用-s或者-server 参数指定Kubernetes API server的地址和端口

## 资源编排

资源清单文件（YAML文件）

- 通过缩进表示层级关系
- 不能使用Tab进行缩进，只能使用空格
- 一般开头缩进两个空格
- 字符后缩进一个空格，比如冒号、逗号等后面
- 使用 --- 表示新的yaml文件开始
- 使用#代表注释

字段说明

- apiVersion API版本
- kind 资源类型
- metadata 资源元数据
- spec 资源规格
- replicas 副本数量
- selector 标签选择器
- template Pod模板
- metadata Pod元数据
- spec Pod规格
- containers 容器配置

快速编排yaml文件

使用kubectl create命令生成yaml文件

```
kubectl create deployment web --image=nginx -o yaml --dry-run > my1.yaml
```

使用kubectl get命令导出yaml文件

```
kutectl get deploy
kubectl get deploy nginx -o=yaml --export > my2.yaml
```

## Pod

### 概念

- 最小部署单元
- 包含多个容器（一组容器的集合）
- 一个pod中容器共享网络命名空间
- pod是短暂的

### Pod的意义

- 创建容器使用docker，一个docker对应是一个容器，一个容器有进程，一个容器运行一个应用程序
- Pod是多进程设计，运行多个应用程序
- 一个Pod有多个容器，一个容器里面运行一个应用程序
- Pod存在为了亲密性应用。两个应用之间进行交互、网络之间调用、两个应用需要频繁调用

### Pod实现机制

- 共享网络 通过Pause容器，把其他业务容器加入到Pause容器里面，让所有业务容器在同一个名称空间中可以实现网络共享。
- 共享存储 引入数据卷概念Volumn，使用数据卷进行持久化存储

### Pod镜像拉取策略

`imagePullPolicy: Always`

- IfNotPresent 默认值，镜像在宿主机上不存在时才拉取
- Always 每次创建Pod都会重新拉取一次镜像
- Never Pod永远不会主动拉取这个镜像

### Pod的资源限制

```
  resource:
    #调度 
    requests:
      memory: "64Mi"
      cpu: "250m"
      # 最大
    limits:
      memory: "128Mi"
      cpu: "500m"
```

### Pod重启机制

​	`restartPolicy: Never`

- Always 当容器终止退出后。总是重启容器，默认策略。
- OnFailure 当容器异常退出（退出状态码非0）时，才重启容器
- Never 当容器终止退出，从不重启容器

### Pod健康检查

livenessProbe  存活检查

如果检查失败，将杀死容器，根据Pod的restartPolicy来操作

readinessProbe 就绪检查

如果检查失败，Kubernetes会把Pod从service endpoints中剔除

Probe支持以下三种检查方式

- httpGet 发送HTTP请求，返回200-400范围状态码为成功
- exec 执行shell命令返回状态码是0为成功
- tcpSocket 发起TCP Socket建立成功

### Pod调度

#### 影响调度的属性

1、Pod资源限制对Pod调用产生影响

```
  resource:
    #调度 
    requests:
      memory: "64Mi"
      cpu: "250m"
```

根据request找到足够node节点进行调度

2、节点选择器标签影响Pod调度

```
spec:
  nodeSelector:
    env_role: dev
```

Pod的额外操作

```
kubectl get pods -o wide # 更多信息

kubectl get nodes k8snode1 --show-labels
```

3、节点亲和性影响Pod调度

- 硬亲和性 	约束条件必须满足
- 软亲和性     尝试满足，不保证
- 反亲和性

支持常用操作符

In	NotIn	Exists	Gt	Lt	DoesNotExists  

4、污点和污点容忍

nodeSelector和nodeAffinity ： Pod调度到某些节点上，Pod属性，调度时候实现

Taint污点：节点不做普通分配调度，是节点属性

场景

- 专用节点
- 配置特点硬件节点
- 基于Taint驱逐

查看节点污点情况

```
kubectl describe node k8smaster | grep Taint
```

污点值

- NoSchedule 一定不被调度
- PreferNoSchedule 尽量不被调度
- NoExecute 不会调度，并且还会驱逐Node已有Pod

为节点添加污点

```
kubectl taint node [node] key=value:污点三个值
```

删除污点

```
kubectl taint node k8snode1 env_role:NoSchedule-
```

污点容忍

```
spec:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
```

## Controller

在集群上管理和运行容器的对象。

Pod是通过Controller实现应用的运维，比如伸缩、滚动升级等。

Pod和Controller之间通过label标签建立关系。

deployment应用场景

- 部署无状态应用
- 管理Pod和ReplicaSet
- 部署、滚动升级等功能
- 应用场景：web服务、微服务

使用deployment部署应用

导出yaml文件

```
kubectl create deployment web --image=nginx --dry-run -o yaml > web.yaml
```

使用yaml部署应用

```
kubectl apply -f web.yaml
```

对外发布（暴露对外端口号）

```
kubectl expose deployment web --port=80 --type=NodePort --target-port=80 --name=web1 -o yaml > web1.yaml

kubectl apply web1.yaml

kubectl get pods,svc
```

应用升级回滚和弹性伸缩

```
# 应用升级
kubectl set image deployment web nginx=nginx:1.15

# 查看升级状态
kubectl rollout status deployment web

# 查看升级版本
kubectl rollout history deployment web

# 回滚到上一个版本
kubectl rollout status deployment web

# 回滚到指定的版本
kubectl rollout undo deployment web --to-revision=2

# 弹性收缩
kubectl scale deployment web --replicas=10
```

### 无状态和有状态

无状态

- 认为Pod都是一样的
- 没有顺序要求
- 不用考虑在哪个node运行
- 随意进行伸缩和扩展

有状态

- 上面因素都考虑到
- 让每个Pod独立的，保持Pod启动顺序和唯一性
- 唯一的网络标识符，持久存储
- 有序，比如mysql主从

deployment和statefueset区别：有身份的（唯一标识的）

根据主机名 + 按照一定规则生成域名

每一个Pod有唯一的主机名

唯一域名：主机名称.service名称.名称空间.svc.cluster.local

### 部署守护进程

DaemonSet

- 在每个node上运行一个pod，新加入的node也同样运行在一个pod里面

### 一次性任务

```
apiVersion: batch/v1
kind: Job
metadata:
	
```

### 定时任务

```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
```



## Service

### Service的意义

- 防止Pod失联（服务发现）
- 定义一组Pod访问策略（负载均衡）

### 常用Service类型

- ClusterIP 集群内部使用
- NodePort 对外访问应用使用
- LoadBalancer 对外访问应用使用，公有云

node内网部署应用，外网一般不能访问到的，使用Nginx反向代理，手动将访问节点加入到Nginx里面。

## ConfigMap

作用：存储不加密数据到etcd，让Pod以变量或者Volume挂载到容器中

场景：配置文件

## k8s集群安全机制

### 概述

1、访问k8s集群时候，需要经过三个步骤完成具体操作

- 认证
- 鉴权（授权）
- 准入控制

2、进行访问时候，过程中都需要经过apiserver，apiserver做统一协调，比如门卫。访问过程中需要证书、token、或者用户名+密码。如果访问pod，需要serviceAccount。

传输安全：对外不暴露8080端口，只能内部访问，对外使用端口6443

认证

客户端身份认证常用方式

- https证书认证，基于ca证书
- http token 认证，通过token识别用户
- http基本认证，用户名+密码认证

鉴权（授权）

- 基于RBAC进行鉴权操作
- 基于角色访问控制

准入控制

- 就是准入控制器的列表，如果列表有请求内容，通过，没有拒绝

### 命名空间

```
# 创建命名空间
kubectl create ns roledemo

# 在新创建的命名空间创建pod
Kubectl run nginx --image=nginx -n roledemo

# 创建角色
kubectl apply -f rbac-role.yaml

kubectl get role -n roledemo
```

