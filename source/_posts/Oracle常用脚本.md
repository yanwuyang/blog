---
title: Oracle常用脚本
date: 2014-04-30 21:28:01
comments: true
categories: Oracle
toc: true 
---

### 建表空间
```sql
create temporary tablespace bjjtxx_space_temp 
tempfile 'D:/oracle/product/10.2.0/oradata/orcl/bjjtxx_temp.dbf' 
size 32m 
autoextend on 
next 32m maxsize 2048m
extent management local; 

create tablespace bjjtxx_space
logging
datafile 'D:/oracle/product/10.2.0/oradata/orcl/bjjtxx.dbf' 
size 32m 
autoextend on 
next 32m maxsize 2048m
extent management local;
```
<!--more-->

### 建用户
```sql
create user bjjtxx_develop identified by bjjtxx
default tablespace bjjtxx_space
temporary tablespace bjjtxx_space_temp;
```
### 用户授权
```sql
grant dba,connect,resource,CTXAPP,create view to bjjtxx_develop;
```

### 锁表查询
***查看锁住的表***
```sql
SELECT b.owner,b.object_name,a.session_id,a.locked_mode  
    FROM v$locked_object a ,dba_objects b  
    WHERE b.object_id = a.object_id; 
```

***查看被锁住的会话 ***
```sql
SELECT b.username,b.sid,b.serial#,logon_time  
    FROM v$locked_object a,v$session b  
    WHERE a.session_id = b.sid order by b.logon_time; 
```

***如果要断开某个会话，执行  ***
```sql
alter system kill session ‘sid,serial#’
```

### 设置表空间大小

***查找出oralce表空间的文件名、路径 ***
```sql
select tablespace_name, file_id, file_name, round(bytes/(1024*1024),0) total_space from dba_data_files; 
```

***修改表空间大小  ***
```sql
ALTER DATABASE DATAFILE 'E:\LARGE.DBF' RESIZE 3000M;
```


***设置表空间最大大小  ***
```sql
ALTER DATABASE DATAFILE ''/oracle/oradata/db/GAME.dbf 
AUTOEXTEND ON NEXT 100M 
MAXSIZE 10000M;
```
