# Kubernetes主要概念梳理

## Pod

pod是k8s调度的最小单元，包含一个或者多个容器（这里的容器你可以暂时认为是docker）。

Pod拥有一个唯一的IP地址，在包含多个容器的时候，依然是拥有一个IP地址，它是怎么办到的呢？

k8s在使用多个容器的时候，用到的就是`共享namespace`，这样Pod里的容器就可以通过localhost通信了，就像两个进程一样。同理的，Pod 可以挂载多个共享的存储卷（Volume），这时内部的各个容器就可以访问共享的 Volume 进行数据的读写。

声明一个Pod，就是写yml文件。一个Pod的yml样例

```
apiVersion: v1           #本版号
kind: Service            #创建的资源类型
metadata:                #元数据必选
  namespace: bcmall      #绑定命名空间
  name: bcmall-srv       #Service资源名称
spec:                    #定义详细信息
  type: NodePort         #类型
  selector:              #标签选择器
    app: container-bcmall-pod 
  ports:                 #定义端口
    - port: 8080         #port 指定server端口,此端口用于集群内部访问
      targetPort: 80     #绑定pod端口
      nodePort: 14000    #将server 端口映射到Node节点的端口,用于外网访问
      protocol: TCP      #端口协议
```

要让这些yml文件生效，你需要用到kubectl 命令，就像这样。

```
kubectl create -f ./bcmall.yaml
```

访问一个Pod，可以通过它的IP，也可以通过内部的域名（这时候就需要CoreDNS）。当这么用的时候，其实Pod的表现，就相当于一台普通的机器，里面的容器就是一堆进程而已。

## 探针和钩子

一个Pod被调度之后，就要进行初始化。初始化肯定是得有一个反馈的，否则都不知道最终有没有启动成功。这些健康检查的功能，叫做探针（Probe）。

常见的有livenessProbe、readinessProbe、startupProbe等三种探针。

`livenessProbe`有点像`心跳`，如果判定不在线了，就会把它干掉；`readinessProbe`一般表示`就绪`状态，也比较像心跳，证明你的服务在正常跑着；`startupProbe`用于判断容器是否已经启动好，避免一些超时等，比如你的JVM启动完毕了，才能对外提供服务。

一般花费120s startupProbe的启动时间，每隔5s检测一下livenessProbe，每隔10s检测一下readinessProbe,是常用的操作。

再说一下钩子（Hook）。主要有`PostStart`和`PreStop`两种。PostStart 可以在容器启动之后就执行，PreStop 则在容器被终止之前被执行。

## 内部组件

- kube-apiserver 提供Rest接口，属于k8s的灵魂，所有的认证、授权、访问控制、服务发现等功能，都通过它来暴露
- kube-scheduler 一看就是个调度组件，实际上它的作用也是这样。它会监听未调度的 Pod，实现你指定的目标
- kube-controller-manager 负责维护整个k8s集群的状态。注意是k8s集群的状态，它不管Pod
- kubelet 这是个守护进程，用来和apiserver通信，汇报自己Node的状态；一些调度命令，也是通过kubelet来接收执行任务
- kube-proxy kube-proxy其实就是管理service的访问入口,包括集群内Pod到Service的访问和集群外访问service。我们上面提到的四种模式，就是通过proxy进行转发的

## 其他概念

- StatefulSet Deployment部署后的实例，它的id都是随机的，比如`bcmall-deployment-5d45f98bd9`，它是无状态的。与此对用的是StatefulSet，生成的实例名称是类似`bcmall-deployment-1`这样的。它具备固定的网络标记，比如主机名，域名等，可以按照顺序来部署和扩展，非常适合类似MySQL这样的实例部署
- DaemonSet 用于确保集群中的每一个节点只运行特定的pod副本，通常用于实现系统级后台任务
- configMap和Secret 顾名思义，就是做配置用的，因为容器或多或少会需要外部传入一些环境变量。可以用来实现业务配置的统一管理， 允许将配置文件与镜像文件分离，以使容器化的应用程序具有可移植性
- PV和PVC 业务运行就需要存储，可以通过PV进行定义。PV的生命周期独立于Pod的生命周期，就是一段网络存储；PVC是用户对于存储的需求：Pod消耗节点资源，PVC消耗PV资源，PVC和PV是一一对应的。没错，它们都是通过yml文件声明的
- StorageClass  可以实现动态PV，是更进一步的封装
- Job 只要完成就立即退出，不需要重启或重建
- Cronjob 周期性任务控制，不需要持续后台运行
- CRD

## 资源限制

k8s提供了`requests`和`limits` 两种类型参数对资源进行预分配和使用限制。

设置

```
requests:
 memory: "64Mi"
 cpu: "250m"
limits:
 memory: "128Mi"
 cpu: "500m"
```

内存的单位是`Mi`，而cpu的单位是`m`，要多别扭有多别扭，但它是有原因的。

`m`是`毫核`的意思。比如，我们的操作系统有4核，把它乘以1000，那就是总CPU资源是4000毫核。如果你想要你的应用最多占用`1/4`核，那就设置成`250m`。



