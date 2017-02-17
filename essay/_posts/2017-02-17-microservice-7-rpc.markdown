---
layout: post
title: "基础服务的设计与RPC"
subtitle: "从SSH单体应用到微服务架构-7"
date: 2017-02-17 12:00:00
author: "shamphone"
header-img: "img/home-bg-post.jpg"
catalog: true
tags: [微服务]

---

>  不少同学询问到如何实施微服务，特别是对项目数量增加的担忧。 在支付渠道设计一文中提到，可以按照渠道来划分项目，一个渠道一个项目，有同学认为这会导致项目太多无法管理。 本文要回答这个问题，在微服务中，我们是如何管理项目的，即微服务的软件过程。 

## 一、RPC技术选型

### Apache Thrift 

### Dubbo  

### Google Protobuf 

## 二、基础服务设计

### 2.1 文件名和编码

rpc接口文件名以 xxx_rpc_service.thrift 来命名；   
protobuf参数文件名以 xxx_service.proto 来命名。 

这两种文件全部使用UTF-8编码

### 2.2 文件结构

文件头需要包含公司版权信息。比如Evernote公司的版权申明：

```hbs

/*
 * Copyright 2007-2016 Evernote Corporation. All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions
 * are met:
 *
 * 1. Redistributions of source code must retain the above copyright
 *    notice, this list of conditions and the following disclaimer.
 * 2. Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 * THIS SOFTWARE IS PROVIDED BY THE AUTHOR ``AS IS'' AND ANY EXPRESS OR
 * IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES
 * OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED.
 * IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY DIRECT, INDIRECT,
 * INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT
 * NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
 * DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
 * THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
 * (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF
 * THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 */


```

### 2.1 接口设计规范

2.1.1 服务名称以 “XXXXService” 的格式命名， XXXX是实体，必须是名词。以下是合理的接口名称：


OrderService， AccountService

### 2.2 参数设计规范


### SDK规范