---
title: Linux常见问题解决方法
---

### 使用SSH登陆Linux的时候提示
 1. 问题描述
```shell
The host '192.168.1.240' is unreachable.
the host may be down,or there may be a problem with the network connection.Sometimes such problems can also be caused by a misconfigured firewall
```
**解决办法**
<!-- more -->
 1. 关闭防火墙
```shell
service iptables stop(重启生效)
iptables -F(立即生效)
```
***如果通过上面的方法还是无法连接那就是sshd服务没有启动***

 2. 启动服务就可以正常访问。
```shell
 service sshd start
```


----------
### connect network is unreachable
原因是没有正确设置IP地址
**解决办法**
 1. 修改ifcfg-eth0
```shell
vi /etc/sysconfig/network-scripts/ifcfg-eth0

service network restart
route add default  gw 192.168.1.1(设置网关)
service network restart
```
----------

### linux从一台主机复杂文件目录到另一台

 1. 复制文件
```shell
scp conf.xml root@192.168.1.240:/usr/conf

说明：scp 文件名 用户名@计算机IP或者计算机名称 :远程路径
```

 2. 复制目录
```shell
scp -r apache-tomcat-6.0.41 root@192.168.1.240:/usr/tomcat

说明：scp -r 目录名 用户名@计算机IP或者计算机名称 :远程路径
```


----------
### make：cc：命令未找到
**解决办法**
 1. 安装gcc
```shell
yum -y install gcc automake autoconf libtool make
```


----------

### /bin/sh:ctags:command not found
**解决办法**
 1. 安装ctags
```shell
yum install ctags
```




