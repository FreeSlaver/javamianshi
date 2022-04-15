---
layout: page
breadcrumb: true
title: MySQL分库分表方案
redirect_from:
  - /2017/08/05/MySQL-Sharding-Approaches.html
category: mysql
categoryStr: MySQL
tags: MySQL sharding
keywords: MySQL sharding stackoverflow
description: 翻译自stack overflow上的一个问答：MySQL sharding approaches，觉得写的非常的好，说的是：为什么不建议做MySql分库分表。
---
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1. MySQL分库分表方案</a>
<ul>
<li><a href="#sec-1-1">1.1. 问题：</a></li>
<li><a href="#sec-1-2">1.2. 回答：</a>
<ul>
<li><a href="#sec-1-2-1">1.2.1. 最好的切分MySQL的方式就是：除非万不得已，否则不要去干它。</a></li>
<li><a href="#sec-1-2-2">1.2.2. 你的SQL语句不再是声明式的（declarative）</a></li>
<li><a href="#sec-1-2-3">1.2.3. 你招致了大量的网络延时</a></li>
<li><a href="#sec-1-2-4">1.2.4. 你失去了SQL的许多强大能力</a></li>
<li><a href="#sec-1-2-5">1.2.5. MySQL没有API保证异步查询返回顺序结果</a></li>
<li><a href="#sec-1-2-6">1.2.6. 总结</a></li>
</ul>
</li>
</ul>
</li>
</ul>
</div>



翻译一个stackoverflow上的问答，关于分库分表的缺点的，原文链接:    
[MySQL sharding approaches?](https://stackoverflow.com/questions/5541421/mysql-sharding-approaches)  


## 问题：<a id="sec-1-1" name="sec-1-1"></a>

什么是最好的MySQL分库分表方案？我想到的有：
1.  应用层切分？
2.  MySQL代理层切分？
3.  提供查找数据库分片的中心服务？

你们知道任何这方面有趣的项目或者工具吗？

## 回答：<a id="sec-1-2" name="sec-1-2"></a>

### 最好的切分MySQL的方式就是：除非万不得已，否则不要去干它。<a id="sec-1-2-1" name="sec-1-2-1"></a>

当你写一个应用时，你通常都想要最快的开发速度。只有必要时，你才开始优化延时，提高吞吐量，  

你切分数据库的原因无非因为**数据库的读或者写**：  
- 数据库写  
写操作永久的超过了服务器的磁盘负载；太多写入导致副本同步永远的落后了。  
- 数据库读  
读取到的数据量太大以至于称爆内存；并且大多数读操作开始直击磁盘而不是从内存中读取数据。  

只有这时，你才需要考虑做数据库切分。  

当你开始切分后，你开始以下面几种方式支付代价：  

### 你的SQL语句不再是声明式的（declarative）<a id="sec-1-2-2" name="sec-1-2-2"></a>

一般的，你用SQL语句告诉数据库你要什么数据，然后让优化器优化SQL，并将SQL转化成数据获取程序。  
这很棒，因为它非常灵活，而且写这些转化程序很无聊，严重影响开发速度。  

分布式环境下，你将A节点的表和B节点的表进行join，甚至有些表的数据大到超过一个节点，  
在A节点和B节点将数据join起来,然后将B节点和C节点join后的数据再次聚合。  
你开始写单方面的hash应用程序来解决这个问题（或者你可以再造MySQL的集群），  
这表示结果你得到一大堆的非声明式的SQL，而且**让程序以一种面向过程的方式开始工作**。  

### 你招致了大量的网络延时<a id="sec-1-2-3" name="sec-1-2-3"></a>

一般的，一条SQL查询语句可以本地解决并且通过优化器以最小的耗时解决这个查询问题。  

在分布式环境下，查询语句必须要通过KV映射，访问多个网络节点（希望是批量访问，  
而不是每个Key都来一次网络往返），或者将WHERE条件放在他们将被执行的节点上。  

但是即使在最好的情况下，涉及到多个网络访问都会更加复杂。特别是MySQL的优化器完全不知道网络延时的情况。  

### 你失去了SQL的许多强大能力<a id="sec-1-2-4" name="sec-1-2-4"></a>

好吧，这或许没那么重要，但是外键约束，其他保证数据完整性的SQL机制，对于跨多个节点是无能为力的。

### MySQL没有API保证异步查询返回顺序结果<a id="sec-1-2-5" name="sec-1-2-5"></a>

当相同类型的数据存放在多个节点上（比如用户数据存放在A,B,C节点上），水平查询需要访问所有节点，  
数据访问时间直接因以节点数线性增长。除非多个节点是以并行方式访问，然后再以Map-Reduce的方式聚合。    

前提是需要提供异步通信的API，但这并不存在于MySQL提供的功能中。可选的方案是在子进程中增加很多的forking和连接。  

### 总结<a id="sec-1-2-6" name="sec-1-2-6"></a>

当你开始做数据库分库分表，数据结构和网络拓扑明显影响到应用的性能。   
为了运行良好，你的应用需要当心这些事情，所以只有应用层的切分才有意义。   

如果需要自动切分，问题会更多（比如决定那个节点的那个列作为hash主键），或者  
你想要手动进行切分，xyz用户去这个主库上，abc去和def去到那个主库上。  

业务功能上的切分有些好处，如果做对了，这对绝大部分开发人员是透明的。因为所有相关的表都存放在本地。   
这让程序的透明性可以从声明式的SQL中尽量受益，并且有更少的网络延时，因为跨节点的网络访问被保持到最小化。  

业务功能上的切分的缺点是：**它不能准许单个表的数据膨胀过大**，这需要设计者的特别注意。  

业务功能切分的好处是：针对一个并没有太多改变的代码库，它相对而言非常简单。    
Booking.com在过去几年进行过几次业务功能上的分库分表，并且一直工作的很好。  

### 相关阅读
[数据库分库分表](/database/MySQL-Sharding.html)   
[使用Maxwell Kafka和Maxwell-Sink进行MySql数据同步](/MySql-ETL-Using-Maxwell-Kafka-MaxwellSink)   
