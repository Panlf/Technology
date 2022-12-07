## Docker发布镜像

1、https://hub.docker.com/ 注册账号

2、在我们的服务器提交自己的镜像

```shell
docker login -u panlf

Login Succeeded
```

3、提交镜像

```shell
docker push 注册用户名/镜像名:版本号
```

