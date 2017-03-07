---
layout:     post
title:      "使用微服务架构重构支付网关"
subtitle:   "从SSH单体应用到微服务架构-4"
date:       2016-09-01 12:00:00
author:     "shamphone"
header-img: "img/home-bg-post.jpg"
catalog:	true
tags:		[微服务]

---

这是微服务重构支付系统的系列文章。 之前的文章请参考：

- [为什么要重构到微服务](http://blog.lixf.cn/essay/2016/08/05/microservice-1/)
- [重构的外部准备工作](http://blog.lixf.cn/essay/2016/08/05/microservice-2/)
- [重构的内部准备工作](http://blog.lixf.cn/essay/2016/08/06/microservice-3/)

在支付系统改进中，我们对原有系统做了整体的评估，选择支付网关作为入手点来进行微服务架构的改进。这里详细介绍我们针对该模块的改进过程，供参考。 

## 原有系统情况

早期启动的时候，对接的支付渠道不多，所有支付渠道和支付网关都实现在一个项目中，部署在一起。其中支付网关是整个项目的核心和入手点。它为各个业务方提供支付全流程的调用接口，签约、代扣、支付、验证，都是通过这个接口来实现的。整个系统使用SSH框架，架构如下：

[![Image of Legacy System](http://blog.lixf.cn/img/in-post/arch-legacy.png)](http://blog.lixf.cn/img/in-post/arch-legacy.png)

业务流程如下：

1. 当接口被调用时， 首先执行参数校验，确认输入的参数的合法性，验证参数签名是否正确。确认过程包括调用账户、用户、支付方式、路由等服务来验证用户ID、账户、支付卡号、支付金额等参数。    
2. 根据输入的支付方式，调用支付路由服务，获取对应的支付渠道。   
3. 调用风控接口进行验证，如果有交易风险，则阻断本次交易。   
4. 生成交易记录；   
5. 调用支付渠道提供的服务执行支付。   
6. 根据支付结果，更新订单状态；  
7. 通知商户订单执行结果。   

在实现上，原有系统实现的类结构图如下：  
[![原有架构](http://blog.lixf.cn/img/in-post/gateway-legacy.jpg)](http://blog.lixf.cn/img/in-post/gateway-legacy.jpg)  

1. 采用SSH架构，支付网关实现为一个大Apache Struts Action类，和支付相关的所有业务逻辑都实现在一个项目中。   
2. 支付网关承载大量的功能，实际上，它是将API网关和业务逻辑都混在在一起实现。 签约、支付、代扣、验证，都在这一个类中实现，代码行数超过1000行，逻辑十分复杂。 
3. 除了风控是进程外调动，其他的服务都是进程内调用，通过springframework来管理各个service。
4. 最终落地调用的支付渠道，是通过抽象的接口来对网关封装渠道的差异。 

[![原有架构-渠道](http://blog.lixf.cn/img/in-post/gateway-channels.jpg)](http://blog.lixf.cn/img/in-post/gateway-channels.jpg)  

最终在这个系统中对接了有30多个渠道，类规模达到2000个。随着业务发展，问题越来越多。高峰期同时有5个渠道在并行开发，还有大量的其他渠道对接问题需要修复。多个人同时修改一个项目代码导致版本控制的工作骤增。上线频发引起服务中断也让业务方很不满。对支付网关的改进是一个循序渐进的过程。这里参考Arun Gupta的[微服务六种设计模式](https://www.javacodegeeks.com/2015/04/microservice-design-patterns.html)，来描述我们所做的改进。 

## 新网关设计 (Chain Pattern)

为了分解旧网关的功能， 我们设计了新的网关。在处理流程上，将其分为三个步骤，采用的是chain模式。 
[![链式模式](http://blog.lixf.cn/img/in-post/pattern-chain.png)](http://blog.lixf.cn/img/in-post/pattern-chain.png)  

> 链式模式，如上图所示，它调用服务A来获取结果，而服务A是通过服务B来交互，B则会和C有交互。 整个过程类似同步的HTTP请求、响应处理。 这其中每个阶段的调用，都是阻塞式的同步调用。每一步都会增加一些业务逻辑处理。 

原支付网关难以维护的一个重要原因是其所承载的功能过多。而签约、支付等操作的实现，却和支付产品有密切的关系。不同的产品，其对应的操作所使用的参数和流程也不一样。我们首先实现了快捷支付产品。新网关接收到请求后，根据用户所选择的支付类型，分发到快捷支付产品接口。快捷支付产品接口调用工行借记卡通道来执行支付， 通道最终落地到工行接口的调用来实现支付。 结果也是按照这个顺序逆向传递会网关接口，并最终发送给调用方。 在这里面，支付网关负责分发、验签等基本功能，支付产品负责参数校验、路由、生成交易记录等功能。最终的支付操作是落地到支付渠道去执行。 

[![链式模式](http://blog.lixf.cn/img/in-post/gateway-chain.jpg)](http://blog.lixf.cn/img/in-post/gateway-chain.jpg)

## 网关拆分(Proxy Pattern)

在快捷支付方式上对这个处理流程验证成功后，我们进一步对网关进行拆分。我们采用完善一个、接入一个的原则，在保留旧网关的功能的同时，开发新的网关和产品。等所有流量都打到新网关上去之后，旧网关就直接废弃了。 为了达到这个目标，我们引入了代理模式：

[![代理模式](http://blog.lixf.cn/img/in-post/pattern-proxy.png)](http://blog.lixf.cn/img/in-post/pattern-proxy.png)  

> 代理模式和聚合模式类似，不同点在于，它会根据业务逻辑需要仅选择一个微服务来调用。

我们首先按照所支持的支付方式，对支付网关做分解，拆分为为网银、快捷、话费、账户、外卡、虚币等支付产品。新网关接口模块是一个proxy，本身并未实现任何业务逻辑，它的工作是将用户请求发送给合适的支付产品去处理。如果这个产品还没有实现，则将其转发到老网关去执行。这样带来的好处是，我们不需要对老网关做任何改动。而且，如果某个支付产品在重构过程中出现问题，我们可以很快切回到老网关去。 

[![代理模式](http://blog.lixf.cn/img/in-post/gateway-proxy.jpg)](http://blog.lixf.cn/img/in-post/gateway-proxy.jpg)

## 支付产品 (Aggregator Pattern)

支付产品是对原有支付网关的业务流程实现的一个重构，按照各个支付产品所支持的功能以及流程来简化原混合在一起的设计。比如快捷支付需要签约和支付，而网银支付则不需要签约。 在支付产品本身的实现上，我们使用的是聚合模式。 

[![聚合模式](http://blog.lixf.cn/img/in-post/pattern-aggregator.png)](http://blog.lixf.cn/img/in-post/pattern-aggregator.png)  

>  聚合的另一种形式是没有页面展示的，它就是一个高层次的微服务组合，供其他服务调用。 在这种情况下，聚合器会从其他的微服务中收集数据，做业务逻辑处理，然后发布成一个服务终端。其他有需要的服务可以调用它。 聚合器设计的要点是要遵循DRY(Don't Repeat Yourself)原则。如果有多个服务需要访问A，B，C服务，那建议的处理方式是，针对这些使用，提炼一个处理逻辑出来，将A、B、C封装为一个新的服务，这个服务可以独立的演化。

支付产品中调用的各个服务，包括支付方式管理， 支付服务管理，支付路由管理、支付记录管理等，都被重构为微服务，在支付产品的实现中，通过Aggregator 模式进行调用。

[![聚合模式](http://blog.lixf.cn/img/in-post/gateway-aggregator.jpg)](http://blog.lixf.cn/img/in-post/gateway-aggregator.jpg)  

在支付产品的流程中，首先需要对参数进行校验，校验成功后，调用风控检查该交易是否可以放行。这两个操作，在处理上可以并行，使用的是分支模式。 
[![分支模式](http://blog.lixf.cn/img/in-post/pattern-branch.png)](http://blog.lixf.cn/img/in-post/pattern-branch.png)  

> 分支模式是聚合模式的扩展，可以允许同时调用两个或者更多的微服务。 

[![分支模式](http://blog.lixf.cn/img/in-post/gateway-branch.jpg)](http://blog.lixf.cn/img/in-post/gateway-branch.png)  

如上，采用分支模式， 使得数据校验和风控可以并发执行。由于风控相对耗时较长，而订单中需要校验的数据较多，这两个操作有必要并发执行。 

## 支付通道 (Aggregator Pattern) 

支付路由根据用户选择的支付方式对支付通道进行筛选，选取合适的支付通道。支付产品调用该通道的接口来最终落地完成支付服务。 每个支付通道对接也被实现为微服务，在支付产品中调用。 

[![聚合模式](http://blog.lixf.cn/img/in-post/gateway-agg-channels.jpg)](http://blog.lixf.cn/img/in-post/gateway-agg-channels.jpg)  

## 通知商户 (Asynchronous Messaging Pattern)

支付产品执行的最后一个步骤是通知调用方支付的结果。 原系统实现是将这个步骤耦合在原有代码中，容易受到调用方接口的稳定性的影响。 为此，这里采用异步消息的模式来进行重构：

[![聚合模式](http://blog.lixf.cn/img/in-post/pattern-messaging.png)](http://blog.lixf.cn/img/in-post/pattern-messaging.png)  

支付产品在完成支付服务后，发出消息到订单消息队列中。 商户回调处理程序接收到消息后，调用商户回调接口告知支付结果。 此外，风控、BI系统等，也可以使用这个消息来同步订单数据。 

[![异步模式](http://blog.lixf.cn/img/in-post/gateway-messaging.jpg)](http://blog.lixf.cn/img/in-post/gateway-messaging.jpg)  