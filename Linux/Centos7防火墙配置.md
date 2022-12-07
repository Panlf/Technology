# Centos7防火墙配置

## 放通某个端口
```shell
firewall-cmd --permanent --zone=public --add-port=5672/tcp
```
## 移除以上规则
```shell
firewall-cmd --permanent --zone=public --remove-port=5672/tcp
```
## 放通某个端口段
```shell
firewall-cmd --permanent --zone=public --add-port=10000-20000/tcp
```
## 查看所有放通的端口
```shell
firewall-cmd --zone=public --list-ports
```
## 查看防火墙的配置
```shell
firewall-cmd --list-all
```
## 放通某个IP访问
```shell
firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source 
address=192.168.1.169 accept'
```
## 移除以上规则
```shell
firewall-cmd --permanent --remove-rich-rule='rule family=ipv4 source address=192.168.1.169 accept'
```
## 放通某个IP段访问
```shell
firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=192.168.2.0/24 accept'
```
## 禁止某个IP访问
```shell
firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=192.168.1.169 drop'
```
## 放通某个IP访问某个端口
```shell
firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=192.168.1.169 port protocol=tcp port=6379 accept'
```
## 重新加载防火墙配置
```shell
firewall-cmd --reload
```
## 关闭防火墙
```shell
systemctl stop firewalld.service
```
## 启动防火墙
```shell
systemctl start firewalld.service
```
## 重启防火墙
```shell
systemctl restart firewalld.service
```
## 查看防火墙状态
```shell
firewall-cmd --state
```