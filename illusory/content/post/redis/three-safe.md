---
title: "Redis入门教程(三)---安全性、事务、发布订阅"
description: "redis发布订阅和事务,感觉基本用不到"
date: 2019-02-24 22:00:00
draft: false
tags: ["Redis"]
categories: ["Redis"]
---

本文主要记录了Redis安全性(设置密码)、Redis事务和Redis提供的的发布与订阅功能。

<!--more-->

> **[Redis系列教程目录](https://www.lixueduan.com/categories/Redis/)**
>
> [Redis入门教程(一)---安装与配置](https://www.lixueduan.com/posts/876962d5.html)
>
> [Redis入门教程(二)---五大基础数据类型与常用命令](https://www.lixueduan.com/posts/8380a4fa.html)
>
> [Redis入门教程(三)---安全性、事务、发布订阅](https://www.lixueduan.com/posts/5df38113.html)
>
> [Redis入门教程(四)---主从复制与持久化](https://www.lixueduan.com/posts/84dd6d72.html)
>
> [Redis入门教程(五)---搭建Redis集群](https://www.lixueduan.com/posts/397ed67.html)
>
> [Redis入门教程(六)---通过JavaApi(Jedis)操作Redis](https://www.lixueduan.com/posts/e456b1e5.html)
>
> [Redis入门教程(七)---通过 Docker 安装 Redis](https://www.lixueduan.com/posts/96375af.html)
>
> .......
>
> 更多文章欢迎访问我的个人博客-->[幻境云图](https://www.lixueduan.com/)

## 1. 安全性

可以为Redis设置密码。
修改配置文件

```shell
[root@localhost etc]# vim redis.conf 

# Warning: since Redis is pretty fast an outside user can try up to
# 150k passwords per second against a good box. This means that you should
# use a very strong password otherwise it will be very easy to break.
#
# requirepass foobared
#设置密码 这里的redis就是密码
requirepass redis

```

重启Redis，并进入客户端，执行查询keys * 提示无权限

```shell
[root@localhost etc]# /usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf
[root@localhost etc]# /usr/local/redis/bin/redis-cli 
127.0.0.1:6379> keys *
(error) NOAUTH Authentication required.

```

输入密码后即可正常使用。

`auth password`

```shell
127.0.0.1:6379> auth redis
OK
127.0.0.1:6379> keys *
1) "zset1"
2) "set1"
3) "list1"

```

也可以在进入客户端时输入密码

```shell
/usr/local/redis/bin/redis-cli -a password
```

不过一般都不设置密码，工作中都是只能内网访问。

## 2. Redis事务

Redis事务非常简单，使用方法如下：

首先使用`multi`打开事务，然后就写数据了，现在设置的数据都会放在队列里进行保存，最后使用`exec`执行，把数据依次存储到Redis中，`discard`取消事务。

`watch` : 监视一个或多个key，如果事务执行前key被修改了，那么事务将被打断。

```shell
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set a 1
QUEUED
127.0.0.1:6379> set b 2
QUEUED
127.0.0.1:6379> set c 3
QUEUED
127.0.0.1:6379> incr a
QUEUED
127.0.0.1:6379> set d a
QUEUED
127.0.0.1:6379> incr d
QUEUED
127.0.0.1:6379> exec
1) OK
2) OK
3) OK
4) (integer) 2
5) OK
6) (error) ERR value is not an integer or out of range

```

不过事务中出错后并不会回滚，前面的还是已经执行了...

## 3. 发布与订阅

Redis提供了简单的发布订阅功能，具体如下：

```shell
# 订阅监听
subscribe [频道]   
# 进行发布消息广播
publish [频道][发布内容]
```

```shell
# 订阅频道cctv
127.0.0.1:6379> subscribe cctv
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "cctv"
3) (integer) 1
# 发布广播
127.0.0.1:6379> publish cctv hello
(integer) 0

# 订阅窗口就可以接受到消息了
1) "message"
2) "cctv"
3) "hello"
```

## 4. 参考

`http://www.runoob.com/redis/redis-hashes.html`

