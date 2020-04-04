---
title: "gRPC入门教程(零)---使用gRPC时遇到的问题"
description: "主要记录在使用gRPC时遇到的一些问题及其解决方案"
date: 2019-05-16 22:00:00
draft: false
tags: ["gRPC"]
categories: ["gRPC"]
---

本文章主要记录在使用`gRPC`时遇到的一些问题及其解决方案。

<!--more-->

> 更多文章欢迎访问我的个人博客-->[幻境云图](https://www.lixueduan.com/)

## 1. 概述

gRPC是支持跨语言的，但是最近在使用`gPRC`跨语言调用时遇到一个奇怪的问题。

## 2. 具体问题

其中客户端由`golang`编写，服务端由`python`编写。

`golang`客户端调用`Python`服务时一直报如下这个错：

```go
rpc error:Code=Unimplemented desc = Method Not Found！
```

下面是官网上的错误列表，其中`GRPC_STATUS_UNIMPLEMENTED`对应的case也是`Method not found on server`。

| Case                                                         | Status code                   |
| :----------------------------------------------------------- | :---------------------------- |
| Client application cancelled the request                     | GRPC_STATUS_CANCELLED         |
| Deadline expired before server returned status               | GRPC_STATUS_DEADLINE_EXCEEDED |
| Method not found on server                                   | GRPC_STATUS_UNIMPLEMENTED     |
| Server shutting down                                         | GRPC_STATUS_UNAVAILABLE       |
| Server threw an exception (or did something other than returning a status code to terminate the RPC) | GRPC_STATUS_UNKNOWN           |

最奇怪的是单独测试时都正常

`python`自己写的客户端 服务端可以正常交互

`golang`写的客户端 服务端也可以正常交互。

但是两个互相调用时会出现`Method Not Found`这个错误

然后都仔细检查代码之后并没有什么问题。

## 3. 原因

看了各种官方文档和`issue`之后也没有找到原因。

最终在下面这个文章里面找到了原因,博主也遇到了相同的问题。

地址：`https://www.itread01.com/content/1547029280.html`

>  `这是由于`.proto文件`中的`package name` 被修改，和 server 端的package 不一致导致的,双方同步`.proto文件` packagename` 重新编译生成对应的代码即可。

由于`python`编写`.proto文件`时没有加`package xxx;`然后`golang`对包名是比较严格的,所以这边加了`package xxx;`就是因为这个小改动导致一直无法正常调用,果然在修改`.proto文件`从新编译后就能成功运行了。

## 4. 参考

`https://www.itread01.com/content/1547029280.html`