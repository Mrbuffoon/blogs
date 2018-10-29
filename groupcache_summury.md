## groupcache源码分析（一）-- 概述
> 正在学习GO语言，基本语法已经差不多掌握，就打算阅读一些经典开源源码吸取营养，groupcache源码代码量不是很大，而且里面可以学到很多缓存相关的知识， 所以选取groupcache源码进行阅读分析。本篇先大概的介绍一下groupcache，然后简单分析一下代码结构，后续将会有一系列文章进行代码层面的详细讲解。

groupcache是memcached作者Brad Fitzpatrick用Go开发的缓存及缓存过滤库，在一定程度上是前者的替代版，现已在Google多个生产环境中投入使用。对比老版本memcached，groupcache去掉了缓存有效期及缓存回收机制，随之而来的是通过自动备份来均衡负载。

相关文档请见[GoDoc](https://godoc.org/github.com/golang/groupcache), github地址：[github](https://github.com/golang/groupcache)

下面先看一下具体代码的结构：

```
consistanthash        实现一致性hash功能

lru                   实现缓存的置换算法（最近最少使用）

singleflight          实现多个同请求的合并，保证“同时”多个同参数的get请求只执行一次操作功能

groupcachepb          grpc生成的代码，用于远程调用

byteview.go           将byte于string进行了一次封装，对外提供不区分两者的接口

groupcache.go         groupcache的主API函数

http.go               http相关的代码

peers.go              单个节点的一些接口实现

sinks.go              暂时没太搞明白

```
基本原理：

我们以通过groupcache查找一个Get（“my”）的过程为例说明：
![示例图片](https://img-blog.csdnimg.cn/20181029182012260.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FqZmd1cmpmbXZ2bHNma2pnbGto,size_16,color_FFFFFF,t_70)

当 GET my 打到 groupcache-1 后：

1、groupcache-1 先看看自己的 cache 里有没有 my，有的话直接返回。

2、要是没有，看看这个请求归不归自己管，若是，去 DataSever 获取，否则问 group-2(假设 my 归 -2管) 要数据，成功返回后 groupcache-1 本地也缓存一份。

3、在 2 过程中，所有后来打到 groupcache-1 的 GET foo 都会阻塞，直到第一个请求返回。


参考链接：

<https://www.csdn.net/article/2013-07-30/2816399-groupcache-readme-go>

<http://orangeholic.iteye.com/blog/2257245>

<http://www.voidcn.com/article/p-kjpxomam-baw.html>

<https://www.jianshu.com/p/f69f3a3a9a78>
