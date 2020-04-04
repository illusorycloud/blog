---
title: "Redis入门教程(四)---主从复制与持久化"
description: "redis主从复制搭建和两种数据持久化方式介绍"
date: 2019-02-26 22:00:00
draft: false
tags: ["Redis"]
categories: ["Redis"]
---

本文主要记录了如何配置Redis主从复制和Redis的持久化机制介绍与具体配置使用，包括RDB和AOF，通过对比阐述了其各自的优缺点。

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



## 1. 主从复制

主节点负责写数据，从节点负责读数据，主节点定期把数据同步到从节点保证数据的一致性。

### 1.1 要点

- 1.Master可以拥有多个slave
- 2.多个Slavic可以连接同一个master外，还可以连接到其他的slave 
- 3.从复制不会阻塞master在同步数据时master可以继续处理client请求 
- 4.提供系统的伸缩性 

### 1.2 过程

- 1.slave与master建立连接，发送sync同步命令
- 2.master开启一个后台进程，将数据库快照保存到文件中，同时master主进程会开始收集新的写命令并缓存
- 3.后台完成保存后，就将文件发送给slave
- 4.slave将此文件保存到硬盘上 

### 1.3 使用

**主服务器不需要任何调整，只需要对从服务器进行配置**。

修改从服务器Redis配置文件：`/usr/local/redis/etc/redis.conf `

```shell
#添加如下配置
#slaveof master服务器IP master服务器端口号
#slaveof <masterip> <masterport>
slaveof 192.168.1.111 6379
```

如果master服务器上的Redis配置了密码，那么还需要配置以下密码

```shell
#masterauth <master-password>
masterauth redis
```

最后使用info查看role角色即可知道是主服务或从服务。

主服务器可读可写，从服务器只能读不能写。

### 1.4 问题

如果从服务器中查看info出现

```shell
master_link_status:down
```

即主从连接失败，这时在Redis.conf配置文件中做如下修改：

```shell
#bindip 表示具有访问权限
#bind 127.0.0.1 即localhost才能访问 修改为0.0.0.0 即都可以访问
bind 0.0.0.0 
```

修改后从重启一下，应该就oK了。

当然可能还要防火墙的问题，需要关掉防火墙

```shell
systemctl stop firewalld # 临时关闭防火墙
systemctl disable firewalld # 禁止开机启动
```

或者主从服务器根本ping不通，这....

## 2. Redis持久化

### 2.1 简介

Redis是一个支持持久化的内存数据库，可以将内存中的数据写入到磁盘里。

Redis有两种持久化机制：

- **RDB**持久化（原理是将Reids在内存中的数据以快照的方式写入到二进制文件dump.rdb中,所以叫RDB），默认开启RDB机制。实际操作过程是fork一个子进程，先将数据集写入临时文件，写入成功后，再替换之前的文件，用二进制压缩存储。 
- **AOF(append-only file)**机制日志的形式记录服务器所处理的每一个写、删除操作，查询操作不会记录以append-only的模式写入一个AOF日志文件中，在redis重启的时候，可以通过回放AOF日志中的写入指令来重新构建整个数据集。 

### 2.2 对比

`RDB方式`是定时保存一次，若突然掉电，很可能会`丢失数据`。

`AOF方式`可以配置每次操作都写入日志文件中,`数据安全性高`。

#### 1. RDB

**优点** ：

1). **单文件备份**: 一旦采用该方式，那么你的整个Redis数据库将只包含一个文件，这对于文件备份而言是非常完美的。

2). **恢复简单**: 对于灾难恢复而言，RDB是非常不错的选择。因为我们可以非常轻松的将一个单独的文件压缩后再转移到其它存储介质上。

3). **性能最大化 **: 对于Redis的服务进程而言，在开始持久化时，它唯一需要做的只是fork出子进程，之后再由子进程完成这些持久化的工作，这样就可以极大的避免服务进程执行IO操作了。

4). 相比于AOF机制，如果数据集很大，RDB的**启动效率会更高**。

**缺点** :

1). **数据丢失** ： 如果你想保证数据的高可用性，即最大限度的避免数据丢失，那么RDB将不是一个很好的选择。因为系统一旦在定时持久化之前出现宕机现象，此前没有来得及写入磁盘的数据都将丢失。

2). **停顿** ： 由于RDB是通过fork子进程来协助完成数据持久化工作的，因此，如果当数据集较大时，可能会导致整个服务器停止服务几百毫秒，甚至是1秒钟。

####  2. AOF

**优点** ： 

1). 该机制可以带来**更高的数据安全性**，即数据持久性。Redis中提供了3中同步策略，即每秒同步、每修改同步和不同步。事实上，每秒同步也是异步完成的，其效率也是非常高的，所差的是一旦系统出现宕机现象，那么这一秒钟之内修改的数据将会丢失。而**每修改同步，即每次发生的数据变 化都会被立即记录到磁盘中**。

2). **安全** ：由于该机制对日志文件的写入操作采用的是append模式，因此在写入过程中即使出现宕机现象，也不会破坏日志文件中已经存在的内容。然而如果我们本次操 作只是写入了一半数据就出现了系统崩溃问题，不用担心，在Redis下一次启动之前，我们可以通过redis-check-aof工具来帮助我们解决数据 一致性的问题。

3). **安全**： 如果日志过大，Redis可以自动启用rewrite机制。即Redis以append模式不断的将修改数据写入到老的磁盘文件中，同时Redis还会创建一个新的文件用于记录此期间有哪些修改命令被执行。因此在进行rewrite切换时可以更好的保证数据安全性。

4). AOF包含一个格式清晰、易于理解的日志文件用于记录所有的修改操作。事实上，我们也可以通过该文件完成数据的重建。

**缺点** ：

1). **大数据时恢复速度慢**，对于相同数量的数据集而言，AOF文件通常要大于RDB文件。

2). 根据同步策略的不同，**AOF在运行效率上往往会慢于RDB**。总之，每秒同步策略的效率是比较高的，同步禁用策略的效率和RDB一样高效。

#### 3. 小结

* **AOF** ： 牺牲一些性能，换取更高的缓存一致性。
* **RDB** ：保证性能最大化，有一定数据丢失风险。

### 2.3 使用

RDB方式是默认开启的不用配置，如果配置开启了AOF那么RDB会关闭。

如何开启AOF？

修改Redis.conf配置文件

```shell
#appendonly no 是否开启aof 改成yes即可
appendonly yes
# The name of the append only file (default: "appendonly.aof")
#设置保存的日志文件名 一般用默认的就好了
appendfilename "appendonly.aof"
```



## 3. 参考

`http://www.runoob.com/redis/redis-hashes.html`

