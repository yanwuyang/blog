---
title: Nginx+Tomcat集群
date: 2014-12-10 14:35:40
comments: true
categories: Tomcat
toc: true 
---
## 环境
|系统	  |  JDK    |Tomcat    |Nginx                                      |
|:-------:|:-------:|:--------:|:-----------------------------------------:|
|CentOS6 192.168.1.231 CentOS6 192.168.1.232  |JDK1.6   |Tomcat6   |nginx-release-centos-6-0.el6.ngx.noarch.rpm|
## 安装JDK
第一步：检查是否已经安装了jdk 存在就卸载 rpm -e 包名
<!--more-->
```
shell rpm -qa | grep java*
```
第二步：查看yum库是否有java安装包
```
shell  yum -y list java*
```
第三步：选择一个进行安装
```
shell yum -y install java-1.6.0-openjdk*
```
第四步：确定是否安装成功
```
shell java -version
java version "1.6.0_20"
OpenJDK Runtime Environment (IcedTea6 1.9.10) (rhel-1.23.1.9.10.el5_7-i386)
OpenJDK Client VM (build 19.0-b09, mixed mode)
显示如上信息表示安装成功
```

## 安装Nginx
***通过yum安装nginx 针对 CentOS系统 其他版本可能会存在差异***

### 下载安装包
```
shell rpm -ivh http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm
```

### 查看nginx信息
```
shell yum info nginx

Loaded plugins: fastestmirror, refresh-packagekit, security
Loading mirror speeds from cached hostfile
 * base: mirrors.yun-idc.com
 * extras: mirrors.yun-idc.com
 * updates: mirrors.yun-idc.com
nginx                                                                    | 2.9 kB     00:00    
nginx/primary_db                                                         |  33 kB     00:00    
Available Packages
Name        : nginx
Arch        : i386
Version     : 1.6.2
Release     : 1.el6.ngx
Size        : 343 k
Repo        : nginx
Summary     : High performance web server
URL         : http://nginx.org/
License     : 2-clause BSD-like license
Description : nginx [engine x] is an HTTP and reverse proxy server, as well as
            : a mail proxy server.
```

### 安装
```
shell yum install nginx
```
### 启动
```
shell service nginx start
```
### 访问

http://ip

查找nginx安装目录
```
shell ps -ef |grep nginx
root      2244     1  0 22:45 ?        00:00:00 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
nginx     2245  2244  0 22:45 ?        00:00:00 nginx: worker process                  
root      2268  1922  0 22:50 pts/1    00:00:00 grep nginx
```
***如果不能连接到nginx，原因很多，但是可以先检查 1,nginx服务是否真的起来了；2,linux服务器防火墙是否打开***

## 安装Tomcat
```
shell yum install tomcat6 tomcat6-webapps tomcat6-admin-webapps
```

### 启动

```
shell service tomcat6 start
```

### 启动

```
shell service tomcat6 stop
```

### 重启

```
shell service tomcat6 restart
```

按照以上方法安装tomcat6默认目录在/usr/share/tomcat6/下
配置文件默认目录在/etc/tomcat6/下

## 集群配置

### 配置nginx.conf
***在http{}中添加***
```
upstream cluster{
 server 192.168.1.231:8080 weight=2;
 server 192.168.1.232:8080 weight=1;
}
```
cluster这个名称可以随便起
weight参数表示权值，权值越高被分配到的几率越大
server 192.168.1.231:8080; 就是一个tomcat节点
***在server{}中添加如下配置 表示对于访问.jsp的页面使用代理服务器跳转到tomcat节点中***
```
location ~ \.jsp$ {
   proxy_pass   http://cluster;
}
```

如果想对根进行设置只需要对

```
location / {
 root   /usr/share/nginx/html;
 index  index.html index.html;
}
修改为
location / {
 proxy_set_header Host $host;
 proxy_pass  http://cluster;
}
即可
```

复杂点的设置如下

```
location / {
 proxy_pass  http://cluster;
 proxy_redirect          off;
 proxy_set_header        Host $host;
 proxy_set_header        X-Real-IP $remote_addr;
 proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
 client_max_body_size    10m;
 client_body_buffer_size 128k;
 proxy_connect_timeout   90;
 proxy_send_timeout      90;
 proxy_read_timeout      90;
 proxy_buffer_size       4k;
 proxy_buffers           4 32k;
 proxy_busy_buffers_size 64k;
 proxy_temp_file_write_size 64k;
}
```

### nginx.conf文件说明
```
#nginx用户
user  nginx;
#nginx进程数字越大对于处理并发能力越强如果设置2就相当于开启了两个nginx
worker_processes  1;
#错误文件日志路径
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;
#对应每个processes允许最大的连接数
events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    #tcp_nopush     on;
    keepalive_timeout  65;
    #gzip  on;
	#负载均衡控制 server 对应集群的一个个节点weight表示权重值数字越大出现的次数越多
	upstream cluster{
	  server 192.168.1.231:8080 weight=1;
	  server 192.168.1.240:8080 weight=2;
	}
	#nginx服务器配置
    server {
		#监听端口
		listen       80;
		#服务器名称
	    server_name  localhost;

		#charset koi8-r;
		#access_log  /var/log/nginx/log/host.access.log  main;
	    #对于请求跟路径时候的处理
		location / {
		   root   /usr/share/nginx/html;
		   index  index.html index.htm;
		   proxy_set_header Host $host;
		   proxy_pass  http://cluster;
		}

	    #error_page  404              /404.html;

	    # redirect server error pages to the static page /50x.html

	    error_page   500 502 503 504  /50x.html;
	    location = /50x.html {
			root   /usr/share/nginx/html;
			#proxy_pass   http://cluster;
		}
	    #设置对于请求jsp时候的处理
	    location ~ \.jsp$ {
			proxy_pass   http://cluster;
		}
	    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
	    #
	    #location ~ \.php$ {
	    #    proxy_pass   http://127.0.0.1;
	    #}

	    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
	    #
	    #location ~ \.php$ {
	    #    root           html;
	    #    fastcgi_pass   127.0.0.1:9000;
	    #    fastcgi_index  index.php;
	    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
	    #    include        fastcgi_params;
	    #}

	    # deny access to .htaccess files, if Apache's document root
	    # concurs with nginx's one
	    #
	    #location ~ /\.ht {
	    #    deny  all;
	    #}
	}
}
```

### Tomcat配置
打开tomcat的文档查看集群说明将集群配置如下 复制到tomcat\conf\server.xml文件 Engine节点下添加如下配置[具体说明查看官方文档](https://tomcat.apache.org/tomcat-6.0-doc/cluster-howto.html)
```xml
 <Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"
			 channelSendOptions="8">

  <Manager className="org.apache.catalina.ha.session.DeltaManager"
		   expireSessionsOnShutdown="false"
		   notifyListenersOnReplication="true"/>

  <Channel className="org.apache.catalina.tribes.group.GroupChannel">
	<Membership className="org.apache.catalina.tribes.membership.McastService"
				address="228.0.0.4"
				port="45564"
				frequency="500"
				dropTime="3000"/>
	<Receiver className="org.apache.catalina.tribes.transport.nio.NioReceiver"
			  address="auto"
			  port="4000"
			  autoBind="100"
			  selectorTimeout="5000"
			  maxThreads="6"/>

	<Sender className="org.apache.catalina.tribes.transport.ReplicationTransmitter">
	  <Transport className="org.apache.catalina.tribes.transport.nio.PooledParallelSender"/>
	</Sender>
	<Interceptor className="org.apache.catalina.tribes.group.interceptors.TcpFailureDetector"/>
	<Interceptor className="org.apache.catalina.tribes.group.interceptors.MessageDispatch15Interceptor"/>
  </Channel>

  <Valve className="org.apache.catalina.ha.tcp.ReplicationValve"
		 filter=""/>
  <Valve className="org.apache.catalina.ha.session.JvmRouteBinderValve"/>

  <Deployer className="org.apache.catalina.ha.deploy.FarmWarDeployer"
			tempDir="/tmp/war-temp/"
			deployDir="/tmp/war-deploy/"
			watchDir="/tmp/war-listen/"
			watchEnabled="false"/>

  <ClusterListener className="org.apache.catalina.ha.session.JvmRouteSessionIDBinderListener"/>
  <ClusterListener className="org.apache.catalina.ha.session.ClusterSessionListener"/>
</Cluster>   
```

```xml
将
<Engine name="Catalina" defaultHost="localhost"> 
添加jvmRoute属性
<Engine name="Catalina" defaultHost="localhost" jvmRoute="node01" > 
注意其他节点JvmRoute要分别不一样
```
在将应用的web.xml 文件末尾添加  distributable元素 如下表示这个应用下的session是可以共享的
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app>
   <distributable/>
</web-app>
```
然后启动服务器发现控制台打印
![控制台打印](003vMReezy6OUXDo7kt61&690.jpg)
```
信息: Manager [localhost#/demo]: skipping state transfer. No members active in cluster group.
一月 04, 2015 7:55:30 下午 org.apache.catalina.tribes.transport.ReceiverBase bind
信息: Receiver Server Socket bound to:/127.0.0.1:4000
一月 04, 2015 7:56:34 下午 org.apache.catalina.ha.session.DeltaManager getAllClusterSessions
警告: Manager [localhost#/demo]: Drop message SESSION-GET-ALL inside GET_ALL_SESSIONS sync phase start date 15-1-4 下午7:55 message date 70-1-1 上午8:00
```

表示没有发现其他的集群中的其他节点。接受Socket bind的IP是127.0.0.1所以我们需要将他改为局域网的IP
将节点Receiver 属性address="auto"改为服务器局域网IP如下；其他节点也要做相应的改动
```xml
<Receiver className="org.apache.catalina.tribes.transport.nio.NioReceiver"
		  address="192.168.1.231"
		  port="4000"
		  autoBind="100"
		  selectorTimeout="5000"
		  maxThreads="6"/>
```
再次启动服务器发现可以找到了其他的节点了

然后启动nginx 访问nginx http://192.168.1.228/demo/分别打印
```
Hello World DF53CB7748D8BC67E0AB8A9BBA13048F.node2
Hello World DF53CB7748D8BC67E0AB8A9BBA13048F.node1
```

可以看出在node2节点设置的 Hello World在node1、node2上分别获取到了表示配置tomcat的session共享已经成功，如果仅是sessionId相同并不能表示session共享成功，因为通过一个相同的IP和端口访问的服务器都会把cookie中的sessionid发送给服务器。

***注意：***
不要单独去访问某一台服务器这样是不能验证session是否配置成功。因为访问单独的服务器在浏览器输入的ip和端口是不一样的，浏览器是不会把访问其他服务器的cookie自动带上发送给服务器。所以通过单独访问每次打印的sessionId肯定是不一样的。