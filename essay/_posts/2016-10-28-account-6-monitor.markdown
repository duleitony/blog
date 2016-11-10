---
layout: post
title: "支付系统的监控与报警"
subtitle: "支付系统设计-6"
date: 2016-10-28 12:00:00
author: "shamphone"
header-img: "img/home-bg-post.jpg"
catalog: true
tags: [支付系统]
---

关于监控，在各个技术网站，几乎都是一搜一大把。几个大的互联网公司，也都有开发自己的监控系统，关于这方面也有不少分享。 这里介绍针对支付系统的监控和报警，但大部分内容，应该来说，对其他系统也是通用的 。

# 监控架构

现在基本上 Zabbix 成为监控的标配了。 一个常规的 Zabbix 监控实现， 是在被监控的机器上部署Zabbix Agent，从日志中收集所需要的数据，分析出监控指标，发送到zabbix服务器上。
[![zabbix 监控](http://blog.lixf.cn/img/in-post/monitor-1.png)](http://blog.lixf.cn/img/in-post/monitor-1.png)

这种方式要求每个机器上部署 Zabbix 客户端，并配置数据收集脚本。如果有数据收集方式有更新，则需要推送到所有的服务器上。 实际上对一个业务来说，大部分系统监控的指标是类似的，而按照这种方式，每个指标在各个被监控系统中还需要单独写脚本实现，工作量大。针对这种情况，可以采用日志集中监控的方式来处理。
日志最终都需要归并到一个日志仓库中，这个仓库可以有很多用途，特别是日常维护中的日志查询工作。多数指标可以在日志上完成计算的。 借助这个系统，也可以完成监控：

[![zabbix 监控](http://blog.lixf.cn/img/in-post/monitor-2.png)](http://blog.lixf.cn/img/in-post/monitor-2.png)

日志使用Apache Flume来收集，通过Apache Kafka来汇总，一般最后日志都归档到Elastic中。 统计分析工作也可以基于Elastic来做，但这个不推荐。 使用Apache Spark 的 Streaming组件来接入Apache Kafka 完成监控指标的提取和计算，将结果推送到Zabbix服务器上，就可以实现可扩展的监控。 这个架构的优势在于：

- 监控脚本的跨系统使用。 指定日志规范后， 只要按照这个规范编制的日志，都可以纳入监控，无需额外配置。

- 服务重新部署时无须考虑监控脚本的部署，所有监控直接生效。

难点在于，提炼一套通用的日志规范，如何考虑如何通过Spark来分析日志。

## 日志收集
Flume和logstash都可以用于日志收集，从实际使用来看，两者在性能上并无太大差异。flume是java系统，logstash是ruby系统。使用中都会涉及到对系统的扩展，这就看那个语言你能hold住了。

## 日志数据流
flume和logstash都支持日志直接入库，即写入hdfs，elastic等，有必要中间加一层kafka吗？太有必要了，日志直接入库，以后分析就限制于这个库里面了。接入kafka后，对于需要日志数据的应用，可以在kafka上做准时时分析，并将结果保存到需要的数据库中。

## 日志分析
Streaming分析，可以走spark，也可以用storm，甚至直接接入kafka做单机处理。这取决于日志数据规模了。spark streaming是推荐的，它集成了多种算法。

# 监控指标

##系统监控
一般系统监控关注如下指标：

-  CPU负载

- 内存使用率；

- 磁盘使用率；

- 网络带宽占用

这些指标在zabbix中
##服务监控
服务监控主要的依赖日志了。
###QPS，每秒请求数
对于使用容器，如tomcat,resin的系统来说，可以从access log中采集到每个接口的qps。没有引入laccess log的系统，考虑通过annotation来规范输出访问计数。

###请求响应时间

- 错误数

## 数据库监控

- 每秒请求数

- 慢查询SQL数
- SQL语句执行时间

## 调用链监控

调用链监控

# 数据库监控
binlog + canal ，将数据库操作同步到zabbix上。

事务监控

# 监控架构

.##日志系统与日志框架
Javaz主流的日志系统有log4j，JULlogback等，日志框架有apache commons logging，slf等，关于这些系统的历史掌故恩怨情仇八卦趣事，网上有不少资料，这里不详细介绍。根据我们的测试，在高并发系统中，关于日志，有如下

# 参考架构
他山之石，可以攻玉

- [大众点评的实时监控系统](http://www.dengshenyu.com/%E5%90%8E%E7%AB%AF%E6%8A%80%E6%9C%AF/2016/09/05/cat.html?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io), 目前已经开源到github成[CAT系统](https://github.com/dianping/cat)了。
- [京东 MySQL 监控之 Zabbix 优化、自动化](http://mp.weixin.qq.com/s?__biz=MzA3MzYwNjQ3NA==&mid=2651296935&idx=1&sn=14ba5d60764be3dd5f3123c625f89c38&scene=0)
- [Mercury:唯品会全链路应用监控系统解决方案详解](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653547643&idx=1&sn=c06dc9b0f59e8ae3d2f9feb734da4459)