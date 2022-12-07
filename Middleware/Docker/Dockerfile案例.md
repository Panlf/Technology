## Dockerfile案例

### 自定义centos7镜像

```dockerfile
# 定义父镜像
FROM centos:7
# 定义作者信息
MAINTAINER i i@163.com
# 执行安全vim命令
RUN yum -y install vim
# 定义默认的工作目录
WORKDIR /usr
# 定义容器启动执行的命令
CMD /bin/bash
```

自定义SpringBoot镜像

```dockerfile
FROM java:8
MAINTAINER i i-i
ADD springboot-hello-0.0.1.jar app.jar
CMD java -jar app.jar
```



```dockerfile
FROM java:8
COPY *.jar /app.jar
CMD ["--server.port=8080"]
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app.jar"]
```





通过dockerfile构建镜像

```shell
docker build -f dockerfile文件路径 -t 镜像名称:版本
```

### 自定义Tomcat镜像

```dockerfile
FROM centos:7
MAINTAINER i i@163.com

COPY readme.txt /usr/local/readme.txt

ADD jdk-8u11-linux-x64.tar.gz /usr/local
ADD apache-tomcat-9.0.22.tar.gz /usr/local

RUN yum -y install vim

ENV MYPATH /usr/local
WORKDIR $MYPATH

ENV JAVA_HOME /usr/local/jdk1.8.0_11
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.22
ENV CATALINA_BASH /usr/local/apache-tomcat-9.0.22
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

EXPOSE 8080
CMD /usr/local/apache-tomcat-9.0.22/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.22/bin/logs/catalina.out
```

