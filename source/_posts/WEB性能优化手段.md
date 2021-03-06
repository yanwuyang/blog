---
title: WEB性能优化手段
date: 2016-12-28
comments: true
categories: WEB
toc: true 
---

## 前端
### 静态资源独立域名
浏览器请求并发数是针对同一域名下的，一般现代浏览器都是6个[了解更多查看](http://www.stevesouders.com/blog/2008/03/20/roundup-on-parallel-connections/)，这样的设计目的其实是对服务器的一种保护，通常使用多个独立域名作为提供网站响应速度的一种有效手段。可以大大拓展这个并发连接数，也就是令浏览器并行下载更多资源，提高站点性能。 具体的实施方案是将静态资源 图片、css、js 放到一个子域名服务器下。这样带来的好处不仅是提供了浏览器并发请求数量还能减少http 头的数据大小。很明显的就是能减少cookie的传输。
<!--more-->
这个浏览器并发请求受限的还有个原因：实则是多个线程创建了socket 与服务器建立了连接。keep-alive技术的存在使得浏览器复用现有连接和服务器通信比创建新连接的性能要更好一些所以，浏览器的并发数其实并不仅仅只是良知的要求，而是双方都需要保护自己的默契，并在可靠的情况下提供更好的性能。

### 缓存
缓存也是性能优化的重要手段之一，这个可以包括服务器常用数据缓存，和静态资源在浏览器中的缓存。

### 资源压缩文件合并
资源压缩技术，文件合并，这样带来的好处就是减少http 请求数和降低网络数据传输包的大小。能有效的提供网站响应数度，
具体的方法有：
1、资源合并：多个图片合并在一个图片中
2、减少请求数：小图片直接使用base64编码

### 资源lazy加载
对于web网页一屏显示不了可以采用lazy加载技术，当请求网页时加载可视区的内容，当拖动滚动条时在动态的加载网页内容。

### 引用资源摆放位置
公用的做法就是对于CSS资源放置在HTML文档的头部，JS资源的应用一般都放在了尾部。或者在script 标签上添加defer。对于引入的第三方没依赖的资源在标签上添加async属性

### CDN加速
将资源放到CDN服务器上

## 后端
### 线程调优
线程数量不是越多越好，适当的线程能充分利用CPU带来性能的提升，但过多的线程因为线程上下文的切换开销返回是性能下降，所以多线程数量和性能会有个最优的值通常是CPU核数的2倍

### 缓存
对于常用的数据加入内存中


## 总结
网站提速的手段方法很多但总结下来无非就是3点：减少请求数量压缩资源大小，充分利用缓存和提高请求并发数。