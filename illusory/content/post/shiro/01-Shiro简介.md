---
title: "Shiro安全框架(一)---什么是Shiro"
description: "RBAC权限控制和Shiro框架介绍"
date: 2019-03-25 22:00:00
draft: false
tags: ["Shiro"]
categories: ["Shiro"]
---

本文主要讲述了什么是RBAC权限控制，同时对Shiro安全框架做了简单介绍。

<!--more-->

> **[Shiro安全框架系列文章目录](https://www.lixueduan.com/categories/Shiro/)**
>
> [Shiro安全框架(一)---什么是Shiro](https://www.lixueduan.com/posts/7dd8d38.html)
>
> [Shiro安全框架(二)---SpringBoot整合Shiro](https://www.lixueduan.com/posts/fd462737.html)
>
>  [Shiro安全框架(三)---Shiro组件详解](https://www.lixueduan.com/posts/f8844037.html)
>
> 源码下载：[GItHub](https://github.com/illusorycloud/springboot-learning)
>
> 更多文章欢迎访问我的个人博客-->[幻境云图](https://www.lixueduan.com/)

## 1. RBAC权限控制

RBAC：`Role-Based Access Control `基于角色的访问控制

### 1.1 RBAC1.0

用户表 角色表 菜单表(权限表)  和两个中间表 用户-角色表 角色-菜单表。

菜单表中存放所有的功能，角色表中设置多种角色(职位)，权限赋给角色，然后在将角色关联到用户上，这样就不用给每个用户都赋值权限了。

![rbac1.0](https://github.com/lixd/blog/raw/master/images/shiro/rbac-base.png)

### 1.2 RBAC2.0

随着项目的扩大，人数特别特别多了，给每个用户赋角色都很麻烦，然后又添加了一个`用户组表`,对用户进行分组，如果角色也特别特别多，那么在加一个`角色组表`，用户组与用户管理，角色组与角色关联，最后用户组再与角色组关联。

![](https://github.com/lixd/blog/raw/master/images/shiro/rbac-usergroup.png)

![](https://github.com/lixd/blog/raw/master/images/shiro/rbac-permission.png)



![rbac2.0](https://github.com/lixd/blog/raw/master/images/shiro/rbac-all.png)



## 2. Shiro

### 2.1 简介

Java中安全管理框架有`spring security`和`shiro`，其中`spring security`依赖于`spring`，且比较复杂，学习曲线比较该，`shiro`比较简单且独立，java se单机环境都可以使用。

### 2.2 名词解释

![](https://github.com/lixd/blog/raw/master/images/shiro/ShiroFeatures_Authentication.png)

**Shiro是一个强大易用的Java安全框架,提供了认证、授权、加密和会话管理等功能**。

- **Authentication**：身份认证/登录，验证用户是不是拥有相应的身份；
- **Authorization**：授权，即权限验证，验证某个已认证的用户是否拥有某个权限；即判断用户是否能做事情，常见的如：验证某个用户是否拥有某个角色。或者细粒度的验证某个用户对某个资源是否具有某个权限；
- **Session Manager**：会话管理，即用户登录后就是一次会话，在没有退出之前，它的所有信息都在会话中；会话可以是普通JavaSE环境的，也可以是如Web环境的；
- **Cryptography**：加密，保护数据的安全性，如密码加密存储到数据库，而不是明文存储；
- **Web Support**：Web支持，可以非常容易的集成到Web环境；
- **Caching**：缓存，比如用户登录后，其用户信息、拥有的角色/权限不必每次去查，这样可以提高效率；
- **Concurrency**：shiro支持多线程应用的并发验证，即如在一个线程中开启另一个线程，能把权限自动传播过去；
- **Testing**：提供测试支持；
- **Run As**：允许一个用户假装为另一个用户（如果他们允许）的身份进行访问；
- **Remember Me**：记住我，这个是非常常见的功能，即一次登录后，下次再来的话不用登录了。

**记住一点，Shiro不会去维护用户、维护权限；这些需要我们自己去设计提供；然后通过相应的接口注入给Shiro即可**。

![](https://github.com/lixd/blog/raw/master/images/shiro/shiro-seq.png)

可以看到：应用代码直接交互的对象是Subject，也就是说Shiro的对外API核心就是Subject；其每个API的含义：

- **Subject**：主体，代表了当前“用户”，这个用户不一定是一个具体的人，与当前应用交互的任何东西都是Subject，如网络爬虫，机器人等；即一个抽象概念；所有Subject都绑定到SecurityManager，与Subject的所有交互都会委托给SecurityManager；可以把Subject认为是一个门面；SecurityManager才是实际的执行者；
- **SecurityManager**：安全管理器；即所有与安全有关的操作都会与SecurityManager交互；且它管理着所有Subject；可以看出它是Shiro的核心，它负责与后边介绍的其他组件进行交互，如果学习过SpringMVC，你可以把它看成DispatcherServlet前端控制器；
- **Realm**：域，Shiro从从Realm获取安全数据（如用户、角色、权限），就是说SecurityManager要验证用户身份，那么它需要从Realm获取相应的用户进行比较以确定用户身份是否合法；也需要从Realm得到用户相应的角色/权限进行验证用户是否能进行操作；可以把Realm看成DataSource，即安全数据源。



### 2.3 具体架构

![](https://github.com/lixd/blog/raw/master/images/shiro/shiro-crchitecture.png)

- **Subject**：主体，可以看到主体可以是任何可以与应用交互的“用户”；
- **SecurityManager**：相当于SpringMVC中的DispatcherServlet或者Struts2中的FilterDispatcher；是Shiro的心脏；所有具体的交互都通过SecurityManager进行控制；它管理着所有Subject、且负责进行认证和授权、及会话、缓存的管理。
- **Authenticator**：认证器，负责主体认证的，这是一个扩展点，如果用户觉得Shiro默认的不好，可以自定义实现；其需要认证策略（Authentication Strategy），即什么情况下算用户认证通过了；
- **Authrizer**：授权器，或者访问控制器，用来决定主体是否有权限进行相应的操作；即控制着用户能访问应用中的哪些功能；
- **Realm**：可以有1个或多个Realm，可以认为是安全实体数据源，即用于获取安全实体的；可以是JDBC实现，也可以是LDAP实现，或者内存实现等等；由用户提供；注意：Shiro不知道你的用户/权限存储在哪及以何种格式存储；所以我们一般在应用中都需要实现自己的Realm；
- **SessionManager**：如果写过Servlet就应该知道Session的概念，Session呢需要有人去管理它的生命周期，这个组件就是SessionManager；而Shiro并不仅仅可以用在Web环境，也可以用在如普通的JavaSE环境、EJB等环境；所有呢，Shiro就抽象了一个自己的Session来管理主体与应用之间交互的数据；这样的话，比如我们在Web环境用，刚开始是一台Web服务器；接着又上了台EJB服务器；这时想把两台服务器的会话数据放到一个地方，这个时候就可以实现自己的分布式会话（如把数据放到Memcached服务器）；
- **SessionDAO**：DAO大家都用过，数据访问对象，用于会话的CRUD，比如我们想把Session保存到数据库，那么可以实现自己的SessionDAO，通过如JDBC写到数据库；比如想把Session放到Memcached中，可以实现自己的Memcached SessionDAO；另外SessionDAO中可以使用Cache进行缓存，以提高性能；
- **CacheManager**：缓存控制器，来管理如用户、角色、权限等的缓存的；因为这些数据基本上很少去改变，放到缓存中后可以提高访问的性能
- **Cryptography**：密码模块，Shiro提高了一些常见的加密组件用于如密码加密/解密的。

## 3. 参考

`https://blog.csdn.net/yangwenxue_admin/article/details/73936803`

`官方文档：http://shiro.apache.org/documentation.html`