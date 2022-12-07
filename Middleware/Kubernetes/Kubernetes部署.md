# Kubernetes部署

## 容器交付流程

开发代码阶段（编写代码、测试、编写Dockerfile）  ----> 持续交付/集成（代码编译打包，制作镜像，上传镜像仓库） ----> 应用部署（环境准备、Pod、Service、Ingress） ----> 运维（监控、故障排查、升级优化）



## K8s部署项目流程

制作镜像（Dockerfiler） ---> 推送镜像仓库 （阿里云）----> 控制器部署镜像 （Deployment） ----> 对外暴露应用（Service、Ingress） ----> 运维（监控、升级）

```
kubectl create deployment javademo1 --image=.... --dry-run -o yaml > javademo1.yaml
```

```
kubectl apply -f javademo1.yaml
```

```
kubectl expose deployment javademo1 --port=8111 --target-port=8111 --type=NodePort
```

