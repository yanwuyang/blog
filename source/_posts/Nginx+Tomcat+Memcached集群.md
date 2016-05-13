---
title: Nginx+Tomcat+Memcached集群
date: 2015-01-14 16:18:41
comments: true
categories: Tomcat
toc: true 
---


Nginx和Tomcat的安装和配置在这里就不说了，不明白的可以看文章[Nginx+Tomcat集群](/2014/12/10/Nginx+Tomcat集群/)。这篇文章是以前面[Nginx+Tomcat集群](/2014/12/10/Nginx+Tomcat集群/)的文章为基础。Nginx+Tomcat+Memcached集群其实很简单，就是将session管理交给了Memcached管理，这样的好处不言而喻，就是如果集群中的节点如果很多的话采用Tomcat节点间的session复制来保证session的共享。这样会给系统带来很大一部分开销。所以就讲session交给了Memcached管理。Memcached是个分布式缓存系统，采用c语言实现。今天我们以一个Memcached为例。对Memcached不在进集群了。
安置Memcached之前需要安装他依赖的软件libevent-devel。
<!--more-->

## 软件安装

### 安装libevent-devel
```
sehll yum install libevent-devel
```

### 安装memcached
```
sehll wget http://memcached.org/files/memcached-1.4.22.tar.gz
sehll tar -zxvf memcached-1.4.22.tar.gz
sehll cd memcached-1.4.22
sehll ./configure && make && make test && make install
```

### 启动memcached
```
shell /usr/local/bin/memcached -d -m 256 -p 11211 -u root -l 192.168.1.233
```
启动memcached的参数不明白的地方可以查看网上的说明

### 下载依赖Jar
***下载memcached session管理的jar包,Tomcat的版本不一样依赖jar包也不一样我这里是以Tomcat6为例并采用javolution方式策略***

memcached-session-manager-1.8.2.jar
spymemcached-2.7.3.jar
msm-xstream-serializer-1.8.2.jar
msm-javolution-serializer-1.8.2.jar
msm-flexjson-serializer-1.8.2.jar
memcached-session-manager-tc6-1.8.2.jar
javolution-5.5.1.jar

将以上包复制到%TOMCAT_HOME%\lib\下
## 配置
修改%TOMCAT_HOME%\conf\context.xml在Context节点下添加如下配置
```xml
<Manager className="de.javakaffee.web.msm.MemcachedBackupSessionManager"
	memcacheNodes="n1:192.168.1.233:11211"
	requestUriIgnorePattern=".*\.(png|gif|jpg|css|js)$"
	sessionBackupAsync="false"
	sessionBackupTimeout="1800000"
	copyCollectionsForSerialization="false"
	transcoderFactoryClass="de.javakaffee.web.msm.serializer.javolution.JavolutionTranscoderFactory"/>
```
要说明的就是memcachedNodes的配置 我上面写的是memcachedNodes="n1:192.168.1.233:11211"
192.168.1.233为memcached所在主机IP地址11211为端口然后如果有多个memcached的话用逗号隔开例如：
memcachedNodes="n1:192.168.1.233:11211,n2:192.168.1.234:11211"

***注意***
每个节点都需要这么配置而且需要保证一样，让后将我们之前在server.xml中的Tomcat集群session复制配置注释掉。