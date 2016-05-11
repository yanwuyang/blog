---
title: Hadoop集群完全分布式部署
date: 2015-01-08 23:45:20 
comments: true 
categories: Hadoop
toc: true
---

## 准备环境
|IP       		 | 主机名称    | 角色                  |操作系统   |
|:--------------:|:-----------:|:---------------------:|:---------:|
|192.168.131.129 |HadoopMaster | NameNode JobTracker   | CentOS 6.6|
|192.168.131.128 |HadoopSlave1 | DataNode TaskTracker  | CentOS 6.6|

<!-- more --> 
## hadoop版本
***hadoop-2.6.0***
***hadoop目录结构***
![hadoop目录结构](003vMReezy6P1MG9JTO50&690.png)

## 修改主机名称
***将192.168.131.129修改为HadoopMaster***
```
shell vi /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=HadoopMaster
```
***将192.168.131.128修改为HadoopSlave1***

```
shell vi /etc/sysconfig/network
HOSTNAME=HadoopSlave
```
***通过命令hostname输出主机名称验证是否修改成功***

```
shell hostname
HadoopMaster
```

## 配置hosts
```
shell vi /etc/hosts
192.168.131.129 HadoopMaster
192.168.131.128 HadoopSlave1
```
***在hosts文件添加如上两行 必须保证所有节点的host一样***


## 安装jdk
第一步：检查是否已经安装了jdk 存在就卸载 rpm -e 包名
```
shell rpm -qa|grep java
```
第二步：查看yum库是否有java安装包
```
shell yum -y list java*
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
```
显示如上信息表示安装成功

## 安装软件（系统一般都自带）
```
shell yum install ssh
shell yum install rsync
```
## 创建hadoop用户
所有节点的用户名称和密码多必须保持一致相同

## 切换到Hadoop用户配置ssh免密码登录
1、在Master节点上执行
```
shell ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa 
shell cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys 
```
2、对authorized_keys 授权
```
shell chmod 744 authorized_keys 
shell ls ~/.ssh/
```
3、验证是否密码登录
```
shell ssh HadoopMaster
```
4、将id_dsa.pub 复制其他Slave节点hadoop用户~/.ssh/目录下
```
shell scp  ~/.ssh/id_dsa.pub hadoop@192.168.1.128:~/.ssh/
```
5、在Slave节点上通过
```
shell cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys 
```
生成authorized_keys并授权
6、在Master节点上验证是否能密码了登录Slave节点
```
shell ssh HadoopSlave1
```

## 切换到root用户下载hadoop

```
shell http://apache.fayea.com/hadoop/common/hadoop-2.6.0/hadoop-2.6.0.tar.gz
```
[hadoop所有版本地址](http://apache.fayea.com/hadoop/common/)

## 解压hadoop
```
shell tar -zxvf hadoop-2.6.0.tar.gz
```
将hadoop移动到/home/hadoop/目录下
把hadoop-2.6.0目录操作权限分配给hadoop用户
```
shell chown hadoop hadoop-2.6.0/*
shell chmod 700 hadoop-2.6.0/*
```
查看文件操作操作用户权限
![文件权限](003vMReezy6P1KnAQkx19&690.png)

## 设置hadoop启动参数
### 设置java安装路径
```
shell vi etc/hadoop/hadoop-env.sh
export JAVA_HOME=java的安装路径
```
如果通过yum安装的jdk不知道java安装路径可以通过如下命令查找
```
shell which java
/use/bin/java
```
得到的路径是/use/bin/java只需要将JAVE_HOME=/use即可
![java环境配置](003vMReezy6P1KHLwtFe7&690.png)

## 配置core-site.xml
```xml
shell /home/hadoop/hadoop-2.6.0/etc/hadoop/core-site.xml

<configuration>
	<property>  
        <name>hadoop.tmp.dir</name>  
        <value>/home/hadoop/tmp</value>  
        <description>Abase for other temporary directories.</description>  
    </property>
	<property>
      <name>fs.default.name</name>
      <value>hdfs://HadoopMaster:9000</value>         
  </property>          
  <property>            
       <name>mapred.job.tracker</name>           
       <value>hdfs://HadoopMaster:9001</value>     
  </property>                      
  <property>  
        <name>io.file.buffer.size</name>  
        <value>4096</value>  
  </property>
</configuration>
```

## 配置hdfs-site.xml
```xml
shell vi /home/hadoop/hadoop-2.6.0/etc/hadoop/hdfs-site.xml

<configuration>
	<property>  
        <name>dfs.nameservices</name>  
        <value>hadoop-cluster1</value>  
    </property>  
    <property>  
        <name>dfs.namenode.secondary.http-address</name>  
        <value>HadoopMaster:50090</value>  
    </property>  
    <property>  
        <name>dfs.namenode.name.dir</name>  
        <value>file:///home/hadoop/dfs/name</value>  
    </property>  
    <property>  
        <name>dfs.datanode.data.dir</name>  
        <value>file:///home/hadoop/dfs/data</value>  
    </property>  
    <property>  
        <name>dfs.replication</name>  
        <value>2</value>  
    </property>  
    <property>  
        <name>dfs.webhdfs.enabled</name>  
        <value>true</value>  
    </property> 
</configuration>
```

## 配置slaves
```
shell vi /home/hadoop/hadoop-2.6.0/etc/hadoop/slaves
```
***集群中所有slave节点的配置文件HadoopSlave1是Slave节点主机名称***
```
shell cat etc/hadoop/slaves
localhost
```
***将localhost修改为HadoopSlave1***

## 配置hadoop安装目录
```
shell vi /etc/profile

export HADOOP_HOME=/home/hadoop/hadoop-2.6.0
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOOME/sbin:$HADOOP_HOME/lib
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"

#使修改立即生效
shell source /etc/profile 
```

***注意：如果不添加上面配置可能会报如下错误***
```
15/01/07 23:52:35 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Starting namenodes on [OpenJDK Client VM warning: You have loaded library /home/hadoop/hadoop-2.6.0/lib/native/libhadoop.so.1.0.0 which might have disabled stack guard. The VM will try to fix the stack guard now.
It's highly recommended that you fix the library with 'execstack -c ', or link it with '-z noexecstack'.
```

## 格式化hadoop
```
shell bin/hadoop namenode -format
```
![格式化hadoop](003vMReezy6P1LyH3gqae&690.jpg)
格式化成功

## 同步节点
将hadoop分别复制到其他节点的相同目录下（例如Master节点hadoop目录是/home/hadoop/hadoop-2.6.0其他节点也是这个目录）
```
shell scp -r hadoop-2.6.0 hadoop@192.168.131.128:/home/hadoop/
```

## 启动hadoop
```
shell sbin/start-all.sh
```

通过http://192.168.131.129:50070查看hadoop相关信息
![启动成功](003vMReezy6P1MlAy3H5d&690.jpg)
如果没有启动成功或者live noades为0查看hadoop启动日志就能很快解决问题
![启动失败](003vMReezy6P1Mzjamje7&690.jpg)

查看日志看到如异常（日志文件在 hadoop logs目录下）
![查看日志](003vMReezy6P1O1zJBg32&690.jpg)
这个问题主要是因为Master节点clusterID 和Slave节点clusterID 号不一致导致解决办法就是删除在core-site.xml中配置的hadoop.tmp.dir 的/home/hadoop/tmp目录下所有文件重新启动就ok了