# Kubernetes集群安装

## kubeadm方式

kubeadm是官方社区推出的一个用于快速部署Kubernetes集群的工具。

```
# 创建一个Master节点
kubeadm init

# 将一个Node节点加入到当前集群中
kubeadm join <Master节点的IP和端口>
```

| 角色   | IP             |
| ------ | -------------- |
| master | 192.168.44.146 |
| node1  | 192.168.44.145 |
| node2  | 192.168.44.144 |



```
#关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

# 关闭selinux
sed -i 's/enforcing/disabled/' /etc/selinux/cofig # 永久
setenforce 0 # 临时

# 关闭swap
swapoff -a # 临时
sed -ri 's/.*swap.*/#&/' /etc/fstab # 永久

# 根据规划设置主机名
hostnamectl set-hostname <hostname>

# 只需master中执行
# 在master添加hosts
cat >> /etc/hosts/ << EOF
192.168.44.146 k8smaster
192.168.44.145 k8snode1
192.168.44.144 k8snode2
EOF

# 将桥接的IPv4流量传递到iptables的链
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system # 生效

# 时间同步
yum install ntpdate -y
ntpdate time.windows.com
```

Kubernetes默认CRI（容器运行时）为Docker，因此先安装Docker

```
> wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
> yum -y install docker-ce-18.06.1.ce-3.el7
> systemctl enable docker & systemctl start docker
> docker --version


> cat > /etc/docker/daemon.json << EOF
{
	"registry-mirrors":["https://b9pmyelo.mirror.aliyuncs.com"]
}
EOF
```

添加阿里云YUM软件源

```
> cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg 
EOF

```

安装kubeadm、kubelet和kubectl

```
yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0
systemctl enable kubelet
```

部署Kubernetes Master

```
kubeadm init \
--apiserver-advertise-address=192.168.44.146 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.18.0 \
--service-cidr=10.96.0.0/12
--pod-network-cidr=10.244.0.0/16
```

使用kubectl工具。init successful会自动出来

master节点执行

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown ${id -u}:${id -g} $HOME/.kube/config

kubectl get nodes
```

​	两个Node节点执行

```
kubeadm join 192.168.44.146:6443 --token ... \
 --discovery-token-ca-cert-hash ...
```

默认token有效期为24小时，当过期之后，该token就不可用了。

```
kubeadm token create --print-join-command
```

部署CNI网络插件

```
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

```
kubectl applu -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl get pods -n kube-system
```

测试

```
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl get pod,svc
```

