---
title: 调试JVM
date: 2021-03-27 22:48:20
comments: true 
categories: JVM
toc: true
---


## 下载OpenJDK7源码

[openjdk-7-fcs-src-b147-27_jun_2011.zip](http://www.java.net/download/openjdk/jdk7/promoted/b147/openjdk-7-fcs-src-b147-27_jun_2011.zip)

<!--more-->
## 解压

```
unzip openjdk-7-fcs-src-b147-27_jun_2011.zip
```

## 安装JDK6
编译openjdk7依赖jdk6，安装jdk6在这就阐述

## 下载安装Apache Ant
https://ant.apache.org/bindownload.cgi


## 编写Build脚本

在hotspot目录下编写构建脚本build.sh

```
#!/bin/bash

export LANG=C

export ALT_BOOTDIR="/usr/java/jdk1.6.0_38"
export ALT_JDK_IMPORT_PATH="/usr/java/jdk1.6.0_38"

export ANT_HOME="/opt/apache-ant-1.9.15"

#允许自动下载依赖包
export ALLOW_DOWNLOADS=true
#使用预编译头文件，以提升便以速度
export USE_PRECOMPILED_HEADER=true

#要编译的版本
export SKIP_DEBUG_BUILD=false
export SKIP_FASTDEBUG_BUILD=true
#包含全部的调试信息
export  ENABLE_FULL_DEBUG_SYMBOLS=1

export HOTSPOT_BUILD_JOBS=3
export ARCH_DATA_MODEL=64
export ALT_OUTPUTDIR=/opt/openjdk/build/hotspot_debug

export CC_INTERP=1

cd make
make jvmg jvmg1 2>&1 | tee /opt/openjdk/build/hotspot_debug.log

```

## 调试JVM

```
#/bin/sh

/opt/openjdk/build/hotspot_debug/linux_amd64_compiler2/jvmg/hotspot -gdb \
-XX:+UseG1GC -XX:+PrintHeapAtGC -XX:+PrintGCDetails -XX:+PrintGC -XX:+TraceMarkSweep -Xloggc:gc.log  Demo

```

## 遇到的问题

https://my.oschina.net/zhangdq/blog/2250314

