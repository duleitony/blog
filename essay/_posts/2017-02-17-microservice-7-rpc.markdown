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


## 一、RPC技术选型

**Apache Thrift** 

**Dubbo**  

**Google Protobuf**



## 二、基础服务设计

基础服务是微服务的服务栈中最底层的模块， 基础服务直接和数据存储打交道，提供数据增删改查的基本操作。

### 2.1 设计规范

**文件规范**

rpc接口文件名以 xxx_rpc_service.thrift 来命名；   
protobuf参数文件名以 xxx_service.proto 来命名。 

这两种文件全部使用UTF-8编码。

**命名规范**

服务名称以 “XXXXService” 的格式命名， XXXX是实体，必须是名词。以下是合理的接口名称。

```hbs
OrderService
AccountService

```

### 2.2 方法设计

由于基础服务主要是解决数据读写问题，所以从使用的角度，对外提供的接口，可以参考数据库操作，标准化为增、删、改、查、统计等基本接口。接口采用 操作+实体来命名，如createOrder。 接口的输入输出参数采用 接口名+Request 和 接口名Response 的规范来命名。 这种方式使得接口易于使用和管理。 

file: xxx_rpc_service.thrift  

```hbs
/**
 * 这里是版权申明
 **/

namespace java com.phoenix.service 
/**
 * 提供关于XXX实体的增删改查基本操作。 
**/
service XXXRpcService {

	/**
	 * 创建实体
	 * 输入参数:
	 *   1. createXXXRequest: 创建请求，支持创建多个实体；
	 * 输出参数
	 *   createXXXResponse: 创建成功，返回创建实体的ID列表；
	 * 异常
	 *  1. userException:		输入的参数有误；
	 *  2. systemExeption:		服务器端出错导致无法创建； 
	 *  3. notFoundException：  必填的参数没有提供。
	 **/
	binary createXXX(1: binary create_xxx_request) throws (1: Errors.UserException userException, 2: Errors.systemException, 3: Errors.notFoundException)


	/**
	 * 更新实体
	 * 输入参数:
	 *   1. updateXXXRequest: 更新请求，支持同时更新多个实体；
	 * 输出参数
	 *   updateXXXResponse: 更新成功，返回被更行的实体的ID列表；
	 * 异常
	 *  1. userException:		输入的参数有误；
	 *  2. systemExeption:		服务器端出错导致无法创建； 
	 *  3. notFoundException：  该实体在服务器端没有找到。
	 **/
	binary updateXXX(1: binary update_xxx_request) throws (1: Errors.UserException userException, 2: Errors.systemException, 3: Errors.notFoundException)

	/**
	 * 删除实体
	 * 输入参数:
	 *   1. removeXXXRequest: 删除请求，按照id来删除，支持一次删除多个实体；
	 * 输出参数
	 *   removeXXXResponse: 删除成功，返回被删除的实体的ID列表；
	 * 异常
	 *  1. userException:		输入的参数有误；
	 *  2. systemExeption:		服务器端出错导致无法创建； 
	 *  3. notFoundException：  该实体在服务器端没有找到。
	 **/
	binary removeXXX(1: binary remove_xxx_request) throws (1: Errors.UserException userException, 2: Errors.systemException, 3: Errors.notFoundException)

	/**
	 * 根据ID获取实体
	 * 输入参数:
	 *   1. getXXXRequest: 获取请求，按照id来获取，支持一次获取多个实体；
	 * 输出参数
	 *   getXXXResponse: 返回对应的实体列表；
	 * 异常
	 *  1. userException:		输入的参数有误；
	 *  2. systemExeption:		服务器端出错导致无法创建； 
	 *  3. notFoundException：  该实体在服务器端没有找到。
	 **/
	binary getXXX(1: binary get_xxx_request) throws (1: Errors.UserException userException, 2: Errors.systemException, 3: Errors.notFoundException)
	
	/**
	 * 查询实体
	 * 输入参数:
	 *   1. queryXXXRequest: 查询条件；
	 * 输出参数
	 *   queryXXXResponse: 返回对应的实体列表；
	 * 异常
	 *  1. userException:		输入的参数有误；
	 *  2. systemExeption:		服务器端出错导致无法创建； 
	 *  3. notFoundException：  该实体在服务器端没有找到。
	 **/
	binary queryXXX(1: binary query_xxx_request) throws (1: Errors.UserException userException, 2: Errors.systemException, 3: Errors.notFoundException)

	/**
	 * 统计符合条件的实体的数量
	 * 输入参数:
	 *   1. countXXXRequest: 查询条件；
	 * 输出参数
	 *   countXXXResponse: 返回对应的实体数量；
	 * 异常
	 *  1. userException:		输入的参数有误；
	 *  2. systemExeption:		服务器端出错导致无法创建； 
	 *  3. notFoundException：  该实体在服务器端没有找到。
	 **/
	binary countXXX(1: binary count_xxx_request) throws (1: Errors.UserException userException, 2: Errors.systemException, 3: Errors.notFoundException)

}

```

### 2.3 异常设计


## 三、服务SDK 


## 四、接口升级

