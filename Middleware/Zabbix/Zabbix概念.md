# Zabbix概念

> 版本 Zabbix 5.x

## 监控


| 监控项| 监控描述 |
| -------- | ------------------------------------------------------------ |
| 硬件监控 | （1）通过远程控制卡：dell的iDRAC，HP的ILO和IBM的IMM等（2）使用IPMI来完成物理设备的监控工作。通常必须要监控就是温度、硬盘故障（3）路由器、交换器、打印机、Windows等 |
| 系统监控 | CPU、内存、硬盘使用率、硬盘IO、系统负载、进程数              |
| 服务监控 | apache、nginx、mysql、memcache、redis、tomcat、jvm、TCP连接数 |
| 性能监控 | 网站性能、服务器性能、数据库性能、存储性能                   |
| 日志监控 | 系统日志、程序访问日志、错误日志、服务运行日志，可以使用ELK进行日志监控 |
| 安全监控 | Nginx+Lua编写了一个WAF通过kibana可以图形化的展示不同的攻击类型的统计。用户登录数，password文件变化、本地所有文件改动 |
| 网络监控 | 端口、WEB、DB、ping包、IDC带宽网络流量、网络流出速率、网络入流量、网络出流量、网络使用率、SMTP |

> Zabbix是由Alexei Vladishev开发的一种网络监视、管理系统，基于Server-Client架构。可用于监控各种网络服务、服务器和网络机器等状态。

## Zabbix功能

- 支持自定义监控脚本，提供需要输出的值即可
- zabbix存储的数据库表结构稍有复杂但是逻辑清晰
- zabbix存在模板的概念，可以方便的将一组监控项进行部署
- zabbix每一个item也就是监控项，都可以看到历史记录，且web界面友好
- zabbix有强大的Trigger（触发器）定义规则，可以定义复杂的报警逻辑
- zabbix提供了ack报警确认机制
- zabbix支持邮件、短信、微信等告警
- zabbix在触发告警后，可以远程执行系统命令
- zabbix有原生的PHP绘图模块

## Zabbix专有词汇

- zabbix server 服务端，收集数据，写入数据
- zabbix agent 部署在被监控的机器上，是一个进程，和zabbix server进行交互，以及负责执行命令
- Host 服务器的概念，指zabbix监控的实体，服务器，交换机等
- Hosts 主机组
- Applications 应用
- Events 事件
- Media 发送通知的通道
- Remote command 远程命令
- Template 模板
- Item 对于某一个指标的监控，称之为Items，如某台服务器的内存使用状况，就是一个item监控项
- Trigger 触发器，定义报警的逻辑，有正常、异常、未知三个状态
- Action，当Trigger符合设定值后，zabbix指定的动作，如发个邮件

## Zabbix程序组件

- Zabbix_server 服务端守护进程
- Zabbix_agentd agent守护进程
- Zabbix_proxy 代理服务器
- Zabbix_database 存储系统，MySQL PGSQL
- Zabbix_web web GUI 图形化界面
- Zabbix_get 命令行工具，测试向agent发起数据采集请求
- Zabbix_sender 命令行工具，测试向server发送数据
- Zabbix_java_gateway java网关

## 部署Zabbix 5.0

```
1、初始化环境、关闭防火墙
systemctl disable --now firewalld

2、zabbix-server 内存尽量更大点，4G为好

3、获取zabbix的下载源
rpm -Uvh https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm

 4、更换zabbix.repo源
 sed -i 's#http://repo.zabbix.com#https://mirrors.aliyun.com/zabbix#' /etc/yum.repos.d/zabbix.repo
 
 5、清空缓存，下载zabbix服务端
 yum clean all
 yum makecache
 
 yum install zabbix-server-mysql zabbix-agent -y
 
 6、安装工具，可以在机器上，使用多个版本的软件，并且不会影响到整个系统的依赖环境
 yum install centos-release-scl -y
 
 7、修改zabbix-front前端源，修改/etc/yum.repos.d/zabbix.repo 启用zabbix前端仓库
 [zabbix-frontend]
 name=Zabbix Official Repository frontend -$basearch
 baseurl=...
 enabled=1 # 开启这里的参数
 gpgcheck=1
 gpgkey=...
 
 8、安装zabbix前端环境，且是安装到scl环境下
 yum install zabbix-web-mysql-scl zabbix-apache-conf-scl -y
 
 9、安装zabbix所需数据库，mariadb
 yum install mariadb-server -y
 
 10、配置数据库，开机启动
 systemctl enable --now mariadb
 
 11、初始化数据库，设置密码
 mysql_secure_installation
 
 12、添加数据库用户，以及zabbix所需的数据库信息
 create database zabbix character set utf8 collate utf_bin;
 
 create user zabbix@localhost identified by '密码';
 
 grant all privileges on zabbix.* to zabbix@localhost;
 
 flush privileges;
 
13、使用zabbix-mysql命令，导入数据库信息
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix 

14、修改zabbix server配置文件，修改数据库的密码
vim /etc/zabbix/zabbix_server.conf
DBPassword=zabbix

15、修改zabbix的php配置文件，修改时区
vim /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf
php_value[date.timezone] = Asia/Shanghai

16、启动zabbix相关服务器
systemctl restart zabbix-server zabbix-agent httpd rh-php72-php-fpm
systemctl enable zabbix-server zabbix-agent httpd rh-php72-php-fpm

17、访问zabbix入口
ip/zabbix

18、安装成功后，默认账号密码
Admin
zabbix
```

## 部署Zabbix客户端

Zabbix5.0版本

agent2新版本采用golang语言开发的客户端

由于是go语言开发，部署起来就很方便了，和之前的程序部署形式不一样了

agent2默认用10050端口，也就是zabbix客户端端口

- 旧版本的客户端 zabbix-agent
- go语言新版客户端 zabbix-agent2

```
1、机器环境准备

2、注意时间正确
yum install ntpdate -y
ntpdate -u ntp.aliyun.com

3、时区的统一配置
mv /etc/localtime{,.bak}
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

具体的zabbix-agent2部署流程

```
rpm -Uvh https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm

 sed -i 's#http://repo.zabbix.com#https://mirrors.aliyun.com/zabbix#' /etc/yum.repos.d/zabbix.repo

yum install zabbix-agent2 -y

# 查看配置文件
vim /etc/zabbix/zabbix_agent2.conf
修改Server 、ServerActive 、 Hostname

# 启动命令
ls -l /usr/sbin/zabbix_agent2
> /usr/sbin/zabbix_agent2

# 启动客户端
systemctl enable --now zabbix-agent2


```

## 验证zabbix-agent2的连通性

```
1、在服务端上通过命令，主动获取数据
yum install zabbix_get -y

zabbix_get -s '监控的ip' -p 10050 -k 'agent.ping'
```

## 解决zabbix-server查看的乱码问题

zabbix默认检测了服务端本身，但是编码有问题

```
1、安装字体
yum -y install wqy-microhei-fonts


2、复制字体
\cp /usr/share/fonts/wqy-microhei/wqy-microhei.tcc /usr/share/fonts/dejavu/DejaVuSans.ttf


```

## 自定义监控内容

自定义监控服务器登录的人数

```
1、明确需要执行的linux命令
who | wc -l

2、手动创建zabbix的配置文件，用于自定义key
vim /etc/zabbix/zabbix_agent2.conf

vim /etc/zabbix/zabbix_agent2.d/userparameter_lgoin.conf
UserParameter=login.user,who | wc -l
```

## 在页面添加zabbix-server的自定义监控项模板

添加流程

- 创建模板
- 创建应用集（好比是一个文件夹，里面放入一堆监控项）
- 创建监控项，自定义Item，你具体想监控的内容
- 创建触发器，当监控项获取到值的时候，进行和触发器比较，判断，决定是否报警
- 创建图形
- 将具体的主机和该模板链接、关联

## 邮件报警

## Zabbix接口

https://www.zabbix.com/documentation/current/en/manual/api

## 监控实施方案

硬件监控

应用服务监控

互联网上有大量的开源模板可以下载使用

>rsync服务监控
>
>​	监控服务器的873端口是存活的
>
>​	有关端口的监控，使用zabbix自带的key net.tcp.port[,873]
>
>​	进行数据推拉，检测效果
>
>监控NFS服务是否正常
>
>​	通过key检测111端口 net.tcp.port[,111]
>
>​	showmount -e ip | wc -l
>
>监控mysql数据库是否正常
>
>​	通过端口 net.tcp.port[,3306]
>
>​	mysql -uroot -p
>
>​	zabbix自带了mysql的监控模板，直接添加主板和mysql的主机关联即可
>
>web服务器监控
>
>​	net.tcp.port[,80]
>
>   zabbix也提供了对web服务器的监控模板

## 监控服务的具体方法

```
zabbix_get -s 'ip' -p 10050 -k 'net.tcp.port[,80]'
```

## 自动发现，自动注册

```
# 准备好服务器
systemctl is-active zabbix-agent2


```

**自动发现**

zabbix server 主动的去发现所有的客户端，然后将客户端的信息，登记在服务端的机器上

缺点是，zabbix-server压力比较大

**自动注册**

zabbix agent2主动上报自己的信息，发给zabbix-server

缺点是agent2可能找不到server（配置文件写错了）

## 分布式监控

> 分布式监控的作用
>
> - 分担server的集中式压力
>   - Agent > proxy > server
> - 多机房之间的网络延时问题

## Zabbix-Proxy

## SNMP监控

简单网络管理协议

```
1、服务端安装snmp监控程序
yum -y install net-snmp net-snmp-utils

2、开启snmp的配置
sed -i.ori '57a view systemview included .1' /etc/snmp/snmp.conf 
systemctl start snmpd.service

3、使用snmp命令
# -v 指定协议版本 -c指定暗号  sysname  snmp的key
snmpwalk -v 2c -c public 172.0.0.1 sysname
```

