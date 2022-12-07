# Kubernetes 命名空间

> Kubernetes 随带许多（ Namespace ）命名空间。一些命名空间很重要，事关你的Kubernetes使用是否正常！搞坏其中一个命名空间即会损坏Kubernetes系统。

这些命名空间包括如下：

- default：默认的命名空间。
- kube-system：系统为对象创建的命名空间。
- kube-public：该命名空间是自动创建的，所有用户（包括未验证身份的用户）都可以读取。该命名空间主要留给集群使用，以防某些资源在整个集群中应该可见、公开可读。这对于提供引导组件所需的集群信息都很有用。它主要由Kubernetes本身来管理。
- kube-node-lease：该命名空间含有与每个节点关联的Lease对象。节点租用允许kubelet发送heartbeat（心跳），以便控制平面能检测节点故障。

即使您不小心删除了Kubernetes系统的所有命名空间，它们也会再度重新生成。这是Kubernetes组件竭力所要做到的。

但有时如果您不走运，删除命名空间在终止阶段卡住，那就没有办法再度重新生成命名空间了。

## default

default命名空间用作您在未指定命名空间的情况下，创建的任何对象的默认位置。

## kube-system

kube-system是Kubernetes中拥有高级权限的对象和服务帐户的命名空间。

Kubernetes控制器的使用源于该命名空间；换句话说，我们会在控制器方面遇到一些问题，在部署新的pods/deployment时可能会出现问题。

不仅如此，该命名空间还包含其他的重要对象，比如kube-dns和kube-proxy，kube-dns是集群域（cluster.local）的权威命名服务器，它递归解析外部名称。不完全限定的短名称（比如myservice）先使用本地搜索路径来完成。

可以在此处（https://cloud.google.com/kubernetes-engine/docs/how-to/kube-dns）和此处（https://www.digitalocean.com/community/tutorials/an-introduction-to-the-kubernetes-dns-service）找到更多的详细信息。

kube-proxy管理这项工作：将发送到集群Kubernetes服务对象的虚拟IP地址，（VIP）的流量转发到适当的后端pod；想了解更多的详细信息，请点击此处（https://www.tigera.io/blog/comparing-kube-proxy-modes-iptables-or-ipvs/）和此处（https://arthurchiao.art/blog/cracking-k8s-node-proxy/）。

这意味着您在解析外部/内部通信时会遇到困难。

## kube-public

kube-public含有一个单一的ConfigMap对象cluster-info，它有助于发现和安全引导。

如果您试图删除所有上述命名空间，服务器会给出如下响应：

```
Errorfrom server (Forbidden): namespaces "kube-public"is forbidden:thisnamespace may not be deleted
```

预计Kubernetes v1.14中添加的kube-node-lease会像任何普通的命名空间一样被删除。

## kube-node-lease

kube-node-lease这个命名空间含有与每个节点关联的Lease对象。节点lease允许kubelet发送heartbeat（心跳），以便控制平面（节点控制器）可以检测节点故障。

那么，如果我们删除了kube-node-lease，会发生什么？Kubernetes通常会为每个节点创建另一个带有Lease对象的对象，但有时命名空间移除操作会在终止状态卡住。

到那时我们会有一个节点Lease，过时的heartbeat可能会告诉节点控制器：该节点访问不了，从而影响节点之间的整体通信。

### 如何修复终止时卡住的命名空间删除？

当然，您可以尝试弄清楚为何命名空间在终止时卡住，但有时您搞不清楚，这时我们可以使用强行删除

创建一个临时JSON文件

```
kubectl getnamespace<terminating-namespace>-o json >tmp.json
```

执行以下命令

```
$ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

编辑您的tmp.json文件。从finalizers字段中删除Kubernetes值，并保存文件。

执行以下命令，更新命名空间

```

curl -k -H "Content-Type: application/json"-X PUT --data-binary @tmp.json
http://127.0.0.1:8001/api/v1/namespaces/<terminating-namespace>/finalize
```

您的输出会像这样子：

```
{
  "kind":"Namespace",
  "apiVersion":"v1",
  "metadata":{
    "name":"<terminating-namespace>",
    "selfLink":"/api/v1/namespaces/<terminating-namespace>/finalize",
    "uid":"b50c9ea4-ec2b-11e8-a0be-fa163eeb47a5",
    "resourceVersion":"1602981",
    "creationTimestamp":"2021-10-18T18:48:30Z",
    "deletionTimestamp":"2021-10-18T18:59:36Z"
  },
  "spec":{

  },
  "status":{
    "phase":"Terminating"
  }}
```

